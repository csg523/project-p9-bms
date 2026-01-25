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
The system shall update the sensor values only for the specific sensor fields present in the received data frame.\
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
The system shall within 50 millisecond disable the charging and discharging outputs if any sensor reading exceeds the configured safety limits.\
Test Intent: Violation is detected if the output current/voltage does not drop to zero when unsafe conditions are simulated.

### SR-2: Alert System
The system shall activate a visual indicator and audible alarm immediately when the system enters the FAULT state to warn the user of unsafe conditions.\
Test Intent: Violation is detected if the visual or audible alarms do not activate upon the system entering the FAULT state.

### SR-3: Stale Sensor Data Handling
The system shall detect missing, stale, or malformed sensor data and transition to a safe state.\
Test Intent: Violation is detected if invalid or missing data does not result in fault detection and transition to a safe state.

### SR-4: Fault Latching
The system shall remain in the FAULT state and prevent operation until a specific manual reset command is received.\
Test Intent: Violation is detected if the system automatically reverts to IDLE/Normal state after the fault condition is removed, without receiving a Reset command.

### SR-5: Power-On Safe State
The system shall initialize into the IDLE state with all power outputs disabled upon system power-up or reset.\
Test Intent: Violation is detected if upon entering power-on state, voltage or current is not zero.

## Non-Functional Requirements

### NFR-1: Safety Response Latency
The system shall change the output state within 50 milliseconds of detecting a safety limit violation.\
Test Intent: Generate a fault condition (step input) and measure the time difference between the signal edge and the output pin changing state using an oscilloscope.

### NFR-2: Alert System Latency
The system shall activate a visual LED indicator and an alarm within 20 ms of entering the FAULT state.
Test Intent: Violation is detected if the LED/Alarm hardware does not toggle to the "ON" state within 20ms of the internal state transitioning to FAULT.

### NFR-3: Logging Reliability
The system shall successfully transmit battery metrics to the cloud with a success rate of >99% when the network is available.
Test Intent: Run the system for 1 hour; violation is detected if more than 1% of expected packets are dropped or malformed.

### NFR-4: Battery Status Indicator
The system shall provide a visual status indicator that reflects the current battery operating mode: charging, discharging, or idle.
Test Intent: Violation is detected if the visual indicator does not display the correct state within 100 ms of the system transitioning between charging, discharging, and idle modes.
