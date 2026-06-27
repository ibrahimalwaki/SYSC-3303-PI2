# Fire Incident Response System

A multi-subsystem fire incident response simulator built in Java for SYSC 3303 (Real-Time Concurrent Systems at Carleton University). The system dispatches autonomous drones to fire zones, tracks their state in real time, and handles hardware faults — all coordinated through concurrent message-passing over UDP.

## Architecture

Three independent subsystems communicate via UDP packets:

```
FireIncidentSubsystem  ──UDP──>  Scheduler  ──UDP──>  DroneSubsystem
       ↑                             |                      |
  reads event CSV              manages zones           runs N drones
                                    |                      |
                               SchedulerGUI  <──── state updates
```

- **FireIncidentSubsystem** — reads fire events from a CSV file and dispatches them to the Scheduler.
- **Scheduler** — assigns drones to zones, tracks progress, and drives the GUI.
- **DroneSubsystem** — manages a pool of drones, each running its own state machine.
- **SchedulerGUI** — live visualization of drone positions, zone status, and fault events.

## Drone state machine

Each drone moves through: `Idle → EnRoute → DroppingAgent → Returning → Idle`

Fault conditions handled mid-flight:
- **Stuck mid-flight** — drone stops moving and is timed out
- **Nozzle jam** — agent cannot be released, drone returns for servicing
- **Packet loss** — detected via sequence numbers; drone is reassigned

## Running the simulation

The three subsystems must be started in separate processes (different run configurations in IntelliJ):

1. Run `Main.java` — starts the Scheduler and GUI
2. Run `DroneSubsystem.java` — starts the drone pool
3. Run `FireIncidentSubsystem.java` — starts feeding events

When all events are processed, a `.log` file is written with timing metrics and per-zone results.

Input files:
- `event_file.csv` — fire events (time, zone, severity, agent required)
- `zone_file.csv` — zone coordinates

## Tests

Open the project in IntelliJ and run any of the JUnit test classes:

| Test class | What it covers |
|---|---|
| `DroneStateMachineJUnitTest` | State transitions and guard conditions |
| `FaultDetectionJUnitTest` | Nozzle jam and stuck-mid-flight scenarios |
| `FireIncidentSubsystemJUnitTest` | Event parsing and dispatch logic |
| `UdpCommunicationJUnitTest` | Packet send/receive correctness |
| `PacketLossDetectionJUnitTest` | Sequence number tracking and loss detection |

## Tech

- Java 17
- POSIX sockets via `java.net.DatagramSocket`
- JUnit 5
- Swing for the GUI
