# Drone Fire Incident Response System

A distributed, real-time fire response simulator built across five iterative milestones for SYSC 3303 (Real-Time Concurrent Systems) at Carleton University. Three independent subsystems coordinate over UDP to dispatch autonomous drones to fire zones, track them through a live GUI, and recover from hardware faults mid-flight.

## System architecture

Three JVM processes communicate exclusively through UDP — no shared memory, no direct method calls between subsystems.

```
FireIncidentSubsystem  ──UDP──>  Scheduler  ──UDP──>  DroneSubsystem
       ↑                             |                      |
  reads event CSV              manages zones           fleet of drones
                                    ↓
                               SchedulerGUI
                         (live map + fault alerts)
```

![Class Diagram](SYSC3303-Project-Group5/PI5ClassDiagram.png)

## Drone state machine

Each drone runs its own state machine. Transitions are driven by messages from the Scheduler and hardware fault signals.

![Drone State Machine](SYSC3303-Project-Group5/P4StateMachine_Drone.png)

Normal flow: `Idle → EnRoute → DroppingAgent → Returning → Idle`

## Fault handling

Three fault types are detected and recovered from automatically:

**Drone stuck mid-flight** — drone position stops updating; Scheduler detects timeout, marks the drone faulted, and reassigns the zone.

![Stuck Fault](SYSC3303-Project-Group5/PI5DroneStuckFault.png)

**Nozzle jam** — fire agent cannot be released; drone returns for servicing and the zone is re-queued.

![Nozzle Jam](SYSC3303-Project-Group5/PI5ZNozzleJameedFault.png)

**Packet loss** — UDP packets are tracked by sequence number; dropped packets trigger retransmission or drone reassignment.

![Packet Loss](SYSC3303-Project-Group5/PI5PacketLoss.png)

## Running the simulation

Each subsystem runs in a separate process. Start them in this order in IntelliJ (or separate terminals):

```
1. Main.java              — Scheduler + GUI
2. DroneSubsystem.java    — drone fleet
3. FireIncidentSubsystem.java — event feed
```

Input files:
- `event_file.csv` — fire events with timestamps, zone IDs, severity, and agent volume
- `zone_file.csv` — zone coordinates and boundaries

A `.log` file with per-zone timing and metrics is written when the simulation ends.

## Tests

Five JUnit 5 test suites cover the core logic:

| Test class | What it covers |
|---|---|
| `DroneStateMachineJUnitTest` | All state transitions and guard conditions |
| `FaultDetectionJUnitTest` | Nozzle jam and stuck-mid-flight detection and recovery |
| `FireIncidentSubsystemJUnitTest` | CSV parsing and event dispatch |
| `UdpCommunicationJUnitTest` | Packet encoding, sending, and receiving |
| `PacketLossDetectionJUnitTest` | Sequence number tracking and loss detection |

## Sequence diagrams

**System initialization**

![Initialization Sequence](SYSC3303-Project-Group5/PI3SequenceDiagram_Initalization.png)

**Drone deployment**

![Deploy Sequence](SYSC3303-Project-Group5/PI3SequenceDiagram_Deploy.png)

**Agent refill**

![Refill Sequence](SYSC3303-Project-Group5/PI3SequenceDiagram_Refill.png)

## Tech

- Java 17
- `java.net.DatagramSocket` — raw UDP communication between subsystems
- Java threads — each drone and subsystem runs concurrently
- JUnit 5 — unit and integration testing
- Swing — live GUI with drone tracking and fault visualization
- Developed iteratively across 5 milestones with team code reviews each iteration
