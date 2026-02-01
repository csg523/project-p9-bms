## System State Model

The high-level system behavior is modeled using a UML state machine.
States represent operational modes of the system.

```mermaid
stateDiagram-v2
    [*] --> POWER_ON
    
    POWER_ON --> INITIALIZATION: System Boot
    
    %% ==========================================
    %% SUPER STATE: INITIALIZATION
    %% Focus: Self-Check & Establishing Comms
    %% ==========================================
    state INITIALIZATION {
        [*] --> SELF_TEST
        
        SELF_TEST --> CONFIG_LOAD: HW Checks OK
        CONFIG_LOAD --> UART_SYNC: Config Loaded
        UART_SYNC --> CHECK_SENSORS: Header/Checksum Found
        CHECK_SENSORS --> VALIDATION_COMPLETE: Readings Plausible

        %% Failure paths within Init
        SELF_TEST --> INIT_FAIL: HW Error
        UART_SYNC --> INIT_FAIL: Timeout (>2s)
        CHECK_SENSORS --> INIT_FAIL: Sensors Mismatch
    }
    
    state init_check <<choice>>
    INITIALIZATION --> init_check
    init_check --> OPERATIONAL: If Init Success
    init_check --> FAULT_HANDLER: If Init Fail



    %% ==========================================
    %% SUPER STATE: OPERATIONAL
    %% Focus: Routine Battery Management
    %% ==========================================
    state OPERATIONAL {
        [*] --> PRE_CHARGE
        
        %% Sub-state: Pre-Charge Logic
        PRE_CHARGE --> STANDBY_IDLE: Bus Voltage Stabilized
        
        %% Sub-state: IDLE
        state STANDBY_IDLE {
            [*] --> MONITORING
            MONITORING --> DEEP_SLEEP: Inactivity (>1hr)
            DEEP_SLEEP --> MONITORING: Wakeup Trigger
        }

        %% Sub-state: CHARGING (Composite)
        state CHARGING {
            [*] --> CC_MODE
            CC_MODE --> CV_MODE: Voltage > Target
            CV_MODE --> BALANCING: Current Tapering
            BALANCING --> CHARGE_COMPLETE: Cells Balanced
            
            
        }

        %% Sub-state: DISCHARGING
        state DISCHARGING {
             [*] --> DISCHARGE_ACTIVE
             DISCHARGE_ACTIVE --> LOW_POWER_WARNING: SoC < 20%
             LOW_POWER_WARNING --> DISCHARGE_ACTIVE: Load Reduced
        }

        %% Operational Transitions
        STANDBY_IDLE --> CHARGING: Current > 100mA
        CHARGING --> STANDBY_IDLE: Current ~ 0mA
        
        STANDBY_IDLE --> DISCHARGING: Current < -100mA
        DISCHARGING --> STANDBY_IDLE: Current ~ 0mA
    }


    %% ==========================================
    %% SUPER STATE: FAULT HANDLING
    %% Focus: Safety & Conservation
    %% ==========================================
    state FAULT_HANDLER {
        [*] --> CLASSIFY_FAULT
        
        state fault_fork <<choice>>
        CLASSIFY_FAULT --> fault_fork
        
        %% Path 1: Soft Fault (Recoverable)
        fault_fork --> SOFT_FAULT: Warning / Timeout
        SOFT_FAULT --> COOLDOWN: Wait for Recovery
        COOLDOWN --> RECOVERY_CHECK: Sensors Normal?
        
        %% Path 2: Hard Fault (Critical)
        fault_fork --> HARD_LOCKOUT: Critical Limits
        HARD_LOCKOUT --> SOS_SIGNAL: Latch State
    }

    %% ==========================================
    %% GLOBAL TRANSITIONS (Safety Net)
    %% ==========================================
    OPERATIONAL --> FAULT_HANDLER: **Safety Trigger**
    
    %% Recovery Transitions
    RECOVERY_CHECK --> OPERATIONAL: Auto-Reset Allowed
    RECOVERY_CHECK --> SOFT_FAULT: Conditions Persist
    SOS_SIGNAL --> INITIALIZATION: **Manual Hard Reset ONLY**
```