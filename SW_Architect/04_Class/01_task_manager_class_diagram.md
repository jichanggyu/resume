# 🧩 task_manager 패키지 – Class Diagram (최종 수정본)

---

# ✅ 1. 클래스 목록 및 구조

### 📘 Interface (추상 클래스)
- `IScenarioReceiver` (상위 시스템으로부터 시나리오 수신 인터페이스)
- `IScenarioExecutor` (시나리오 실행 추상화 인터페이스)

### 📘 Core Classes
- `TaskManagerNode` (rclpy.Node, 메인 ROS2 노드 역할)
- `ScenarioLibrary` (시나리오 정의 및 저장소)
- `ScenarioExecutionManager` (시나리오 실행 관리)
- `CommandBroker` (command_executor 통신 담당)
- `ExecutionStateTracker` (시나리오 실행 상태 추적)

### 📘 Scenario Classes
- `PickPlaceScenario` (implements `IScenarioExecutor`)
- `DoorOperationScenario` (implements `IScenarioExecutor`)
- `CustomScenario` (implements `IScenarioExecutor`)

---

# ✅ 2. 인터페이스 상세 설명

## 📘 IScenarioReceiver
- **책임**: 상위 시스템(ACS, API 서버 등)으로부터 시나리오 요청 수신
- **행동**:
  - `+ receive_scenario_request(request_data: Dict) -> ScenarioRequest`
  - `+ validate_request(request_data: Dict) -> bool`
- **협력**: TaskManagerNode, 상위 시스템

## 📘 IScenarioExecutor
- **책임**: 특정 유형의 시나리오 실행 로직 정의
- **행동**:
  - `+ execute() -> bool`
  - `+ get_command_sequence() -> List[Command]`
  - `+ on_command_completion(result: CommandResult) -> NextAction`
  - `+ pause() -> bool`
  - `+ resume() -> bool`
  - `+ cancel() -> bool`
- **협력**: ScenarioExecutionManager

---

# ✅ 3. Core Class 상세 설명

## 📘 TaskManagerNode
- **책임**: 메인 ROS2 노드, 외부 요청 수신 및 처리
- **속성**:
  - `- scenario_library: ScenarioLibrary`
  - `- execution_manager: ScenarioExecutionManager`
  - `- command_broker: CommandBroker`
  - `- state_tracker: ExecutionStateTracker`
- **행동**:
  - `+ handle_scenario_request(scenario_type: str, params: Dict) -> bool`
  - `+ cancel_current_scenario() -> bool`
  - `+ get_execution_status() -> ExecutionStatus`
  - `+ initialize() -> bool`
- **협력**: ScenarioLibrary, ScenarioExecutionManager, CommandBroker
- **비고**: ROS2 서비스 및 토픽 관리

## 📘 ScenarioLibrary
- **책임**: 시스템에서 지원하는 시나리오 정의 및 관리
- **속성**:
  - `- available_scenarios: Dict[str, ScenarioDefinition]`
  - `- scenario_dependencies: Dict[str, List[str]]`
- **행동**:
  - `+ get_scenario(scenario_type: str) -> IScenarioExecutor`
  - `+ register_scenario(scenario_type: str, executor_class: Type[IScenarioExecutor]) -> bool`
  - `+ check_scenario_availability(scenario_type: str) -> bool`
- **협력**: TaskManagerNode
- **비고**: 다양한 시나리오 유형 지원, 동적 등록 가능

## 📘 ScenarioExecutionManager
- **책임**: 시나리오 실행 및 상태 관리
- **속성**:
  - `- current_scenario: IScenarioExecutor`
  - `- command_broker: CommandBroker`
  - `- state_tracker: ExecutionStateTracker`
  - `- execution_state: ExecutionState`
- **행동**:
  - `+ execute_scenario(scenario: IScenarioExecutor, params: Dict) -> bool`
  - `+ handle_command_result(result: CommandResult) -> bool`
  - `+ cancel_execution() -> bool`
  - `+ pause_execution() -> bool`
  - `+ resume_execution() -> bool`
- **협력**: CommandBroker, ExecutionStateTracker
- **비고**: 시나리오 실행 흐름 제어

## 📘 CommandBroker
- **책임**: command_executor 노드와의 통신 담당
- **속성**:
  - `- command_clients: Dict[str, ROS2Client]`
  - `- pending_commands: Queue[CommandRequest]`
  - `- last_result: Dict[str, CommandResult]`
- **행동**:
  - `+ send_command(command_type: str, params: Dict) -> CommandID`
  - `+ cancel_command(command_id: CommandID) -> bool`
  - `+ check_command_status(command_id: CommandID) -> CommandStatus`
  - `+ wait_for_result(command_id: CommandID, timeout: float) -> CommandResult`
- **협력**: ScenarioExecutionManager, command_executor_node (외부)
- **비고**: 비동기 명령 실행 지원

## 📘 ExecutionStateTracker
- **책임**: 시나리오 실행 상태 추적 및 보고
- **속성**:
  - `- current_scenario: str`
  - `- current_step: int`
  - `- total_steps: int`
  - `- start_time: DateTime`
  - `- status: ExecutionStatus`
- **행동**:
  - `+ update_state(state: Dict) -> None`
  - `+ publish_state() -> None`
  - `+ log_execution_progress() -> None`
  - `+ get_current_state() -> ExecutionStatus`
- **협력**: ScenarioExecutionManager, 상태 모니터링 시스템
- **비고**: 실시간 상태 발행

---

# ✅ 4. 시나리오 클래스 상세 설명

## 📘 PickPlaceScenario
- **책임**: 물체 픽앤플레이스 작업 시나리오 구현
- **속성**:
  - `- steps: List[CommandDefinition]`
  - `- current_step: int`
  - `- object_info: ObjectInfo`
  - `- destination: Position`
- **행동**:
  - `+ execute() -> bool`
  - `+ get_command_sequence() -> List[Command]` 
  - `+ on_command_completion(result: CommandResult) -> NextAction`
- **협력**: ScenarioExecutionManager
- **비고**: 비전 데이터 기반 동적 명령 생성 가능

## 📘 DoorOperationScenario
- **책임**: 문 개폐 작업 시나리오 구현
- **속성**:
  - `- steps: List[CommandDefinition]`
  - `- current_step: int`
  - `- door_type: DoorType`
- **행동**:
  - `+ execute() -> bool`
  - `+ get_command_sequence() -> List[Command]`
  - `+ on_command_completion(result: CommandResult) -> NextAction`
- **협력**: ScenarioExecutionManager
- **비고**: 문 유형별 동작 차별화

## 📘 CustomScenario
- **책임**: 사용자 정의 시나리오 지원
- **속성**:
  - `- command_sequence: List[CommandDefinition]`
  - `- current_index: int`
  - `- error_handling_policy: ErrorPolicy`
- **행동**:
  - `+ execute() -> bool`
  - `+ get_command_sequence() -> List[Command]`
  - `+ on_command_completion(result: CommandResult) -> NextAction`
  - `+ load_from_definition(definition: Dict) -> bool`
- **협력**: ScenarioExecutionManager
- **비고**: 동적 시나리오 정의 지원

---

# ✅ 5. 클래스 간 관계 요약 (UML 관계)

- TaskManagerNode → (Aggregation) → ScenarioLibrary, ScenarioExecutionManager, CommandBroker, ExecutionStateTracker
- ScenarioExecutionManager → (Aggregation) → CommandBroker, ExecutionStateTracker
- ScenarioExecutionManager → (Association) → IScenarioExecutor
- ScenarioLibrary → (Creation) → IScenarioExecutor 구현체
- PickPlaceScenario, DoorOperationScenario, CustomScenario → (Inheritance) → IScenarioExecutor
- CommandBroker → (Association) → command_executor_node (외부)

---

# ✅ 6. ROS2 및 시스템 특성 반영

- TaskManagerNode는 ROS2 Node로 동작
- 주요 ROS2 Services:
  - `/task_manager/execute_scenario`
  - `/task_manager/cancel_scenario`
  - `/task_manager/pause_scenario`
  - `/task_manager/resume_scenario`
  - `/task_manager/get_status`
- 주요 ROS2 Topics:
  - `/task_manager/execution_status` (발행)
  - `/task_manager/execution_progress` (발행)
- 상위 시스템과의 통신:
  - REST API 엔드포인트 (FastAPI)
  - WebSocket 이벤트 (ACS 인터페이스)

---

# ✅ 7. 예외/에러 상황 대응

| 상황 | 대응 방법 |
|:--|:--|
| 존재하지 않는 시나리오 요청 | 오류 반환 및 로깅 |
| 시나리오 실행 중 에러 | 1) 재시도 2) 대체 명령 3) 안전 취소 |
| command_executor 응답 없음 | 타임아웃 후 실행 취소 |
| 상위 시스템 연결 끊김 | 현재 작업 완료 또는 안전 정지 |
| 시나리오 실행 취소 요청 | 현재 명령 취소 및 정리 작업 수행 |

---

# ✅ 8. task_manager와 command_executor, h/w node 역할 구분

| task_manager | command_executor | h/w node |
|:--|:--|:--|
| 상위 시스템으로부터 시나리오 유형 수신 | 시나리오 실행을 위한 개별 명령 처리 | 하드웨어 드라이버와 통신 |
| 시나리오 실행 흐름 관리 | 명령별 실행 로직 제공 | 하드웨어 수준 명령 변환 및 실행 |
| 명령 순서 결정 및 조정 | 개별 명령의 성공/실패 처리 | 하드웨어 상태 모니터링 |
| 시나리오 성공/실패 결정 | 하위 레벨 명령 시퀀스 실행 | 안전 기능 제공 |
| 복합 명령 시퀀스 관리 | 명령 실행 피드백 제공 | 센서 데이터 수집 및 제공 |

---

# 📢 최종 요약

- 상위 시스템으로부터 시나리오 요청을 받아 처리하는 구조 확립
- 시나리오 실행을 위한 명령 순서 관리 및 조정
- command_executor와의 명확한 역할 분담 (시나리오 vs 개별 명령)
- 다양한 시나리오 타입 지원을 위한 확장 가능한 구조
- 실시간 실행 상태 추적 및 보고
- 상위 시스템과 하위 시스템 간의 효과적인 중개자 역할

---
