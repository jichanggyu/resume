# 🧩 command_executor 패키지 – Class Diagram (최종 수정본)

---

# ✅ 1. 클래스 목록 및 구조

### 📘 Interface (추상 클래스)
- `ICommandHandler` (모든 명령어 실행 핸들러 인터페이스)
- `IHardwareConnector` (하드웨어 노드 연결 인터페이스)

### 📘 Core Classes
- `CommandExecutorNode` (rclpy.Node, 명령 수신 및 실행 관리)
- `CommandDispatcher` (명령어 분배 및 핸들러 호출)
- `CommandExecutionContext` (명령 실행 상태 저장)
- `HardwareNodeConnector` (H/W 노드와 통신)
- `CommandResponsePublisher` (명령 실행 결과 발행)

### 📘 CommandHandler Implementations
- `PickCommandHandler` (implements `ICommandHandler`)
- `PlaceCommandHandler` (implements `ICommandHandler`)
- `MoveCommandHandler` (implements `ICommandHandler`)
- `GripperOpenCommandHandler` (implements `ICommandHandler`)
- `GripperCloseCommandHandler` (implements `ICommandHandler`)
- `DoorOpenCommandHandler` (implements `ICommandHandler`)
- `DoorCloseCommandHandler` (implements `ICommandHandler`)
- `StopCommandHandler` (implements `ICommandHandler`)

---

# ✅ 2. 인터페이스 상세 설명

## 📘 ICommandHandler (Interface)
- **책임**: 특정 명령을 실행하기 위한 로직 정의
- **행동**:
  - `+ execute(context: CommandExecutionContext) -> CommandResult`
  - `+ validate_params(params: CommandParams) -> bool`
  - `+ cancel() -> bool`
- **협력**: CommandDispatcher
- **에러처리**: 실행 실패 시 적절한 오류 코드 반환
- **비고**: 취소 요청 지원 필수

## 📘 IHardwareConnector (Interface)
- **책임**: H/W 노드와의 통신 추상화
- **행동**:
  - `+ send_hw_command(command_type: str, params: Dict) -> HWCommandID`
  - `+ get_hw_status(command_id: HWCommandID) -> HWStatus`
  - `+ cancel_hw_command(command_id: HWCommandID) -> bool`
- **협력**: 각 CommandHandler 구현체
- **에러처리**: 통신 오류 적절히 처리
- **비고**: 다양한 하드웨어 인터페이스 지원 가능

---

# ✅ 3. Core Class 상세 설명

## 📘 CommandExecutorNode
- **책임**: task_manager로부터 명령 수신 및 실행 조율
- **속성**:
  - `- dispatcher: CommandDispatcher`
  - `- hw_connector: HardwareNodeConnector`
  - `- current_context: CommandExecutionContext`
  - `- response_publisher: CommandResponsePublisher`
- **행동**:
  - `+ handle_command(command_type: str, params: Dict) -> CommandID`
  - `+ check_status(command_id: CommandID) -> CommandStatus`
  - `+ cancel_command(command_id: CommandID) -> bool`
- **협력**: CommandDispatcher, HardwareNodeConnector
- **비고**: ROS2 서비스 제공, 동시 명령 관리

## 📘 CommandDispatcher
- **책임**: 명령 유형에 따라 적절한 핸들러에 위임
- **속성**:
  - `- handlers: Dict[str, ICommandHandler]`
  - `- active_commands: Dict[CommandID, ICommandHandler]`
- **행동**:
  - `+ dispatch(command_type: str, params: Dict) -> CommandID`
  - `+ register_handler(command_type: str, handler: ICommandHandler) -> bool`
  - `+ get_active_command(command_id: CommandID) -> ICommandHandler`
- **협력**: ICommandHandler 구현체들
- **비고**: 핸들러 동적 등록 지원

## 📘 CommandExecutionContext
- **책임**: 명령 실행 컨텍스트 및 상태 관리
- **속성**:
  - `- command_id: CommandID`
  - `- command_type: str`
  - `- params: Dict`
  - `- hw_command_ids: List[HWCommandID]`
  - `- status: CommandStatus`
  - `- result: CommandResult`
- **행동**:
  - `+ update_status(status: CommandStatus) -> None`
  - `+ set_result(result: CommandResult) -> None`
  - `+ add_hw_command(hw_command_id: HWCommandID) -> None`
- **협력**: ICommandHandler, HardwareNodeConnector
- **비고**: 명령 생명주기 전체 데이터 관리

## 📘 HardwareNodeConnector
- **책임**: H/W 노드와의 통신 관리
- **속성**:
  - `- hw_clients: Dict[str, ROS2Client]`
  - `- pending_commands: Dict[HWCommandID, HWCommandStatus]`
- **행동**:
  - `+ send_command(hw_node: str, command: str, params: Dict) -> HWCommandID`
  - `+ check_command_status(command_id: HWCommandID) -> HWCommandStatus`
  - `+ cancel_command(command_id: HWCommandID) -> bool`
- **협력**: 모든 CommandHandler, H/W 노드(외부)
- **비고**: 다양한 H/W 노드 지원 (로봇 팔, 그리퍼, 센서 등)

## 📘 CommandResponsePublisher
- **책임**: 명령 실행 결과 및 상태 발행
- **속성**:
  - `- status_publisher: ROS2Publisher`
  - `- result_publisher: ROS2Publisher`
- **행동**:
  - `+ publish_status(command_id: CommandID, status: CommandStatus) -> None`
  - `+ publish_result(command_id: CommandID, result: CommandResult) -> None`
- **협력**: CommandExecutorNode
- **비고**: 비동기 상태 업데이트 지원

---

# ✅ 4. CommandHandler 구현체 상세 설명

## 📘 PickCommandHandler
- **책임**: Pick 명령 실행 로직
- **속성**:
  - `- hw_connector: IHardwareConnector`
  - `- execution_steps: List[HWCommand]`
- **행동**:
  - `+ execute(context: CommandExecutionContext) -> CommandResult`
  - `+ create_pick_sequence(params: Dict) -> List[HWCommand]`
- **협력**: HardwareNodeConnector
- **비고**: 비전 시스템 기반 좌표 계산 및 다단계 명령 실행

## 📘 PlaceCommandHandler
- **책임**: Place 명령 실행 로직
- **속성**:
  - `- hw_connector: IHardwareConnector`
  - `- execution_steps: List[HWCommand]`
- **행동**:
  - `+ execute(context: CommandExecutionContext) -> CommandResult`
  - `+ create_place_sequence(params: Dict) -> List[HWCommand]`
- **협력**: HardwareNodeConnector
- **비고**: 안전한 물체 배치 로직 포함

## 📘 MoveCommandHandler
- **책임**: 로봇 이동 명령 실행 로직
- **속성**:
  - `- hw_connector: IHardwareConnector`
  - `- movement_calculator: MovementCalculator`
- **행동**:
  - `+ execute(context: CommandExecutionContext) -> CommandResult`
  - `+ validate_coordinates(x: float, y: float, z: float) -> bool`
- **협력**: HardwareNodeConnector
- **비고**: 다양한 이동 모드 지원 (조인트, 카테시안 등)

## 📘 GripperOpenCommandHandler
- **책임**: 그리퍼 열기 명령 실행
- **속성**:
  - `- hw_connector: IHardwareConnector`
- **행동**:
  - `+ execute(context: CommandExecutionContext) -> CommandResult`
  - `+ calculate_open_position(width: float) -> Dict`
- **협력**: HardwareNodeConnector
- **비고**: 다양한 폭 설정 지원

## 📘 GripperCloseCommandHandler
- **책임**: 그리퍼 닫기 명령 실행
- **속성**:
  - `- hw_connector: IHardwareConnector`
  - `- force_settings: Dict[str, float]`
- **행동**:
  - `+ execute(context: CommandExecutionContext) -> CommandResult`
  - `+ calculate_grip_force(object_type: str) -> float`
- **협력**: HardwareNodeConnector
- **비고**: 물체에 따른 적응형 그립력 조절

## 📘 DoorOpenCommandHandler
- **책임**: 문 열기 명령 실행
- **속성**:
  - `- hw_connector: IHardwareConnector`
  - `- door_profiles: Dict[str, DoorProfile]`
- **행동**:
  - `+ execute(context: CommandExecutionContext) -> CommandResult`
  - `+ select_door_opening_strategy(door_type: str) -> DoorStrategy`
- **협력**: HardwareNodeConnector
- **비고**: 다양한 문 유형 지원

## 📘 DoorCloseCommandHandler
- **책임**: 문 닫기 명령 실행
- **속성**:
  - `- hw_connector: IHardwareConnector`
  - `- door_profiles: Dict[str, DoorProfile]`
- **행동**:
  - `+ execute(context: CommandExecutionContext) -> CommandResult`
  - `+ select_door_closing_strategy(door_type: str) -> DoorStrategy`
- **협력**: HardwareNodeConnector
- **비고**: 안전한 문 닫기 동작 구현

## 📘 StopCommandHandler
- **책임**: 긴급 정지 명령 실행
- **속성**:
  - `- hw_connector: IHardwareConnector`
  - `- priority: int`
- **행동**:
  - `+ execute(context: CommandExecutionContext) -> CommandResult`
  - `+ stop_all_hw_commands() -> bool`
- **협력**: HardwareNodeConnector, CommandDispatcher
- **비고**: 최우선 처리, 모든 활성 명령 취소

---

# ✅ 5. 클래스 간 관계 요약 (UML 관계)

- CommandExecutorNode → (Aggregation) → CommandDispatcher, HardwareNodeConnector, CommandResponsePublisher
- CommandDispatcher → (Association) → ICommandHandler 구현체들
- 모든 CommandHandler → (Inheritance) → ICommandHandler
- HardwareNodeConnector → (Realization) → IHardwareConnector
- 모든 CommandHandler → (Association) → HardwareNodeConnector
- CommandExecutorNode → (Creation) → CommandExecutionContext

---

# ✅ 6. ROS2 및 시스템 특성 반영

- CommandExecutorNode는 ROS2 Node로 동작
- task_manager로부터 명령 수신을 위한 서비스:
  - `/command_executor/execute_command`
  - `/command_executor/check_status`
  - `/command_executor/cancel_command`
- H/W node 통신을 위한 클라이언트:
  - `/hw_node/move_arm`
  - `/hw_node/control_gripper`
  - `/hw_node/check_status`
- 명령 상태 및 결과 토픽:
  - `/command_executor/command_status`
  - `/command_executor/command_result`
- 평균 명령 처리 주파수: 50Hz

---

# ✅ 7. 예외/에러 상황 대응

| 상황 | 대응 방법 |
|:--|:--|
| 잘못된 명령 매개변수 | 유효성 검증 후 오류 반환 |
| 지원하지 않는 명령 유형 | 적절한 오류 코드와 메시지 반환 |
| H/W node 응답 없음 | 타임아웃 후 실패 처리, 재연결 시도 |
| 명령 실행 중 취소 요청 | 즉시 H/W 명령 취소 후 상태 업데이트 |
| H/W node 오류 응답 | 오류 코드 변환 후 task_manager에 전달 |

---

# ✅ 8. task_manager와 command_executor, hw_node 역할 구분

| task_manager | command_executor | hw_node |
|:--|:--|:--|
| 시나리오 유형을 선택하고 관리 | 개별 명령을 H/W 명령으로 변환 | 하드웨어 직접 제어 |
| 전체 작업 흐름 제어 | 다단계 H/W 명령 시퀀스 생성 | 저수준 하드웨어 명령 실행 |
| 시나리오 간 전환 결정 | 명령 실행 상태 모니터링 | 센서 데이터 수집 및 처리 |
| 상위 시스템 요청 처리 | H/W 노드 오류 처리 및 재시도 | 안전 기능 구현 |
| UI/사용자 피드백 제공 | 완료/실패 결과 생성 및 반환 | 하드웨어 상태 모니터링 |

---

# 📢 최종 요약

- task_manager로부터 명령을 수신하고 H/W node로 전달하는 중개자 역할 명확화
- 각 명령 유형별 전용 핸들러로 책임 분리
- 하드웨어 통신 추상화로 다양한 H/W 지원
- 명령 실행 상태 추적 및 결과 피드백 시스템 구현
- 안전하고 유연한 명령 실행 체계
- 표준화된 오류 처리 및 예외 상황 대응