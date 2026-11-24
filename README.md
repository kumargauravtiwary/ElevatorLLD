# ElevatorLLD
1. Problem statement
Design an elevator management system for a multi-floor building containing multiple elevators.

The system should support:
- Multiple elevators operating independently.
- External requests (Hall Calls): passengers request UP or DOWN from a floor.
- Internal requests (Car Calls): passengers select a destination floor inside an elevator.
- Intelligent elevator dispatching using Nearest Elevator and SCAN algorithms.
- Dynamic elevator assignment based on current position, direction, capacity and pending requests.
- Door operations and safety mechanisms.
- Elevator movement, direction and operational states.
- Handling concurrent requests from multiple floors.
2. Functional requirements
ID	Requirement	Description
FR1	Multiple elevators	Support N elevators
FR2	Hall requests	UP/DOWN requests from any floor
FR3	Car requests	Select destination from inside elevator
FR4	Dispatching	Assign optimal elevator to requests
FR5	Movement	Move between adjacent floors
FR6	Door control	Open, close and obstruction handling
FR7	Direction	UP, DOWN, IDLE
FR8	Capacity	Prevent overloading
FR9	Maintenance	Disable elevators for maintenance
FR10	Safety	Emergency stop and alarm

3. Non-functional requirements
- Scalability: Support multiple buildings and elevators.
- Concurrency: Handle simultaneous requests safely.
- Reliability: Avoid losing accepted requests.
- Extensibility: Allow new dispatch algorithms.
- Maintainability: Separate elevator movement, dispatching and door management.
- Safety: Never move while doors are open.
4. Assumptions and scope
For this design:
- Building has floors numbered 0 to N-1.
- Building contains multiple elevators.
- All elevators serve all floors.
- Each elevator has a maximum passenger capacity.
- Elevator movement is modeled one floor at a time.
- A request can be assigned to only one elevator.
- Dispatching uses a configurable strategy: Nearest or SCAN-based.
- Hardware integration, physical motor control and fire-service protocols are outside the initial software simulation scope.

5. Core object-oriented design
5.1 Main entities
Class	Responsibility
Building	Maintains floors and elevators
Floor	Represents an individual floor
Elevator	Maintains elevator state and operations
ElevatorController	Coordinates elevator operations
Request	Base class for elevator requests
HallRequest	External UP/DOWN request
CarRequest	Internal destination request
Dispatcher	Assigns requests to elevators
DispatchStrategy	Defines elevator selection algorithm
NearestElevatorStrategy	Selects the nearest suitable elevator
ScanStrategy	Selects an elevator using directional scheduling
Door	Manages elevator door operations
Display	Shows floor and direction
ElevatorScheduler	Manages pending requests
SafetyController	Handles overload, obstruction and emergency conditions

REST API design
For a distributed elevator management system, the following APIs can expose the operations.
Method	Endpoint	Purpose
POST	/api/v1/hall-requests	Submit external request
POST	/api/v1/elevators/{id}/requests	Submit internal request
GET	/api/v1/elevators	Get all elevator statuses
GET	/api/v1/elevators/{id}	Get elevator details
GET	/api/v1/requests/{id}	Get request status
POST	/api/v1/elevators/{id}/maintenance	Enable maintenance
DELETE	/api/v1/elevators/{id}/maintenance	Disable maintenance
POST	/api/v1/elevators/{id}/emergency-stop	Trigger emergency handling

Database design
For a persistent elevator management platform, consider the following tables.
Elevator
Column	Type	Description
elevator_id	BIGINT	Primary key
building_id	BIGINT	Building reference
current_floor	INT	Current position
direction	VARCHAR	UP/DOWN/IDLE
state	VARCHAR	Operational state
capacity	INT	Maximum capacity
current_load	INT	Current passengers

Elevator_Request
Column	Type	Description
request_id	BIGINT	Primary key
request_type	VARCHAR	HALL/CAR
source_floor	INT	Request floor
destination_floor	INT	Nullable for hall requests
direction	VARCHAR	UP/DOWN
elevator_id	BIGINT	Assigned elevator
status	VARCHAR	Request status
created_at	TIMESTAMP	Request timestamp

Elevator_Stop
Column	Type	Description
stop_id	BIGINT	Primary key
elevator_id	BIGINT	Elevator reference
floor_number	INT	Scheduled stop
direction	VARCHAR	Direction
sequence_number	INT	Execution sequence
status	VARCHAR	PENDING/COMPLETED

For reliability, persist request assignments and state transitions. The actual movement state should be reconciled with the elevator's hardware telemetry rather than trusting database state alone.

Concurrency and thread safety
Elevator requests may arrive simultaneously from multiple floors.

Consider this situation:
- Passenger A presses UP at floor 2.
- Passenger B presses DOWN at floor 7.
- Passenger C selects floor 9 inside elevator 1.
Potential issues:
1. Two dispatchers assign the same elevator based on stale state.
2. Duplicate requests get scheduled.
3. An elevator moves while another thread opens its door.
4. Requests are lost during application restart.
Recommended approach:
Component	Concurrency mechanism
Dispatcher	Centralized synchronized assignment or lock
Elevator state	Atomic state transitions / per-elevator lock
Stop queues	Thread-safe ordered collections or protected collections
Request assignment	Transactional assignment with version checks
Event processing	Idempotent event handlers
Persistence	Database transactions and optimistic locking

For a Java implementation, a useful architecture is:
- One command-processing queue for incoming requests.
- One logical scheduler per elevator.
- One dispatcher responsible for hall-call allocation.
- Thread-safe communication between dispatcher and elevator schedulers.
Avoid using a single global lock for every elevator because it unnecessarily limits parallel execution.
16. Design patterns used
Pattern	Usage	Benefit
Strategy	Nearest / SCAN dispatch	Switch algorithms easily
State	Elevator and door states	Controlled state transitions
Observer	Elevator status updates	Update display panels
Factory	Create elevator/request objects	Centralized object creation
Command	Represent requests as commands	Queue, retry and audit
Singleton	Optional building controller	Single coordinator per building

elevator-system/
│
├── src/main/java/com/example/elevator/
│   │
│   ├── controller/
│   │   ├── ElevatorController.java
│   │   └── RequestController.java
│   │
│   ├── model/
│   │   ├── Building.java
│   │   ├── Floor.java
│   │   ├── Elevator.java
│   │   ├── Door.java
│   │   └── Request.java
│   │
│   ├── enums/
│   │   ├── Direction.java
│   │   ├── ElevatorState.java
│   │   └── RequestStatus.java
│   │
│   ├── strategy/
│   │   ├── DispatchStrategy.java
│   │   ├── NearestElevatorStrategy.java
│   │   └── ScanStrategy.java
│   │
│   ├── service/
│   │   ├── Dispatcher.java
│   │   ├── ElevatorScheduler.java
│   │   ├── RequestService.java
│   │   └── SafetyController.java
│   │
│   ├── repository/
│   │   ├── ElevatorRepository.java
│   │   └── RequestRepository.java
│   │
│   └── ElevatorApplication.java
│
└── src/test/java/
    └── com/example/elevator/
        ├── DispatcherTest.java
        ├── ElevatorTest.java
        ├── ScanStrategyTest.java
        └── SafetyControllerTest.java