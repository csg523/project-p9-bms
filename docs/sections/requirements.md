## Stakeholder Requirements

Stakeholders require a system that:

- Functions as a working embedded safety supervisor.
- Receives simulated battery sensor data over a UART interface.
- Enables or disables charging and discharging based on readings from the sensors.
- Publishes battery metrics and safety-related status to a remote server using MQTT.
- Responds to changes in environmental conditions within a bounded time.
- Transitions to a safe state upon observing abnormal conditions.

## Functional Requirements

### FR-1 Normal Battery Operations
The system shall regulate the charging and discharging output to match the requested voltage and current setpoints.\
Test Intent: Violation is detected if the measured output voltage/current deviates from the setpoint by more than the allowed tolerance.

### FR-2 Internal Sensor Value Update
The system shall update the sensor values only for the specific sensor fields present in the received UART frame.\
Test Intent: Violation is detected if the system changes the state upon receiving an empty data frame.

### FR-3 Stale Data Handling
The system shall determine whether battery operation is safe based on the most recent available sensor information.\
Test Intent: Violation is detected if unsafe sensor conditions occur without the system declaring an unsafe operating state.

### FR-4 State Transititons
The system shall transition between IDLE, CHARGE, DISCHARGE and FAULT modes, depending upon environment conditions and external control instruction.\
Test Intent: Violation is detected if the state does not change upon a valid external command, or if the system fails to enter FAULT mode when unsafe conditions are simulated.

### FR-5 Data Logging
The system shall publish battery metrics and safety status to the remote server at a periodic, configurable interval.\
Test Intent: Violation is detected if the time difference between two consecutive upload packets exceeds the configured interval.

## Safety Requirements

### SR-1 Threshold Protection
The system shall immediately disable the charging and discharging outputs if any sensor reading exceeds the configured safety limits.\
Test Intent: Violation is detected if the state does not change to DISCHARGING mode when unsafe conditions are simulated.

### SR-2: Communication Timeout
The system shall transition to the FAULT state if valid sensor data has not been received for longer than the defined timeout period.\
Test Intent: Violation is detected if upon halting the data stream, system does not enter FAULT state.

### SR-3: Stale Sensor Data Handling
The system shall detect missing, stale, or malformed sensor data and transition to a safe state.\
Test Intent: Violation is detected if invalid or missing data does not result in fault detection and transition to a safe state.

### SR-4: Fault Latching
The system shall remain in the FAULT state and prevent operation until a specific manual reset command is received.This prevents oscillation between states if a fault is intermittent, which causes stress to battery chemistry and contactors.\
Test Intent: Violation is detected if after simulating conditions, system reverts back to normal state.

### SR-5: Power-On Safe State
The system shall initialize into the IDLE state with all power outputs disabled upon system power-up or reset. This prevents the system from accidentally conducting current immediately after a reboot before safety checks are complete and helps reducing in battery health deterioration.\
Test Intent: Violation is detected if upon entering power-on state, voltage or current is not zero.

## Non-Functional Requirements

### NFR-1: Safety Response Latency
The system shall change the output state within 50 milliseconds of detecting a safety limit violation.
Test Intent: Generate a fault condition (step input) and measure the time difference between the signal edge and the output pin changing state using an oscilloscope.

### NFR-2: Alert System
The system shall activate a visual LED indicator and an alarm when entering the FAULT state, within 20 ms of state transition.
Test Intent: Violation is detected if the LED/Alarm hardware does not toggle to the "ON" state, when the system is in FAULT state, within 20ms.

### NFR-3: Cloud Logging
The system shall publish battery metrics and system status to the cloud at an interval of 10 seconds.
Test Intent: Violation is detected if a new data payload is not received within the 10-second window.
