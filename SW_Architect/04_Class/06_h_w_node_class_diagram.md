# 🧩 h_w_node 패키지 – Class Diagram (최종 수정본)

---

# ✅ 1. 클래스 목록 및 구조

### 📘 Interface (추상 클래스)
- `IHardwareController` (하드웨어 제어 인터페이스)
- `IRobotDriver` (로봇 드라이버 인터페이스)
- `ICameraDriver` (카메라 드라이버 인터페이스)
- `ISensorDriver` (센서 드라이버 인터페이스)
- `IIOController` (I/O 제어 인터페이스)
- `IStatePublisher` (상태 발행 인터페이스)
- `IHeartbeatGenerator` (하트비트 생성 인터페이스)

### 📘 Core Classes
- `HardwareNode` (rclpy.Node, 하드웨어 노드)
- `HardwareManager` (하드웨어 관리)
- `RobotControllerManager` (로봇 컨트롤러 관리)
- `SensorManager` (센서 관리)
- `IOManager` (I/O 관리)
- `HardwareStatePublisher` (하드웨어 상태 발행)
- `HardwareCommandSubscriber` (하드웨어 명령 구독)
- `DriverFactory` (드라이버 팩토리)
- `HeartbeatManager` (하트비트 관리)

### 📘 Data Classes
- `HardwareState` (하드웨어 상태 데이터)
- `JointPosition` (관절 위치 데이터)
- `SensorReading` (센서 읽기 데이터)
- `IOState` (I/O 상태 데이터)
- `RobotCommand` (로봇 명령 데이터)
- `HardwareConfiguration` (하드웨어 구성 데이터)
- `HeartbeatSignal` (하트비트 신호 데이터)

---

# ✅ 2. 인터페이스 상세 설명

## 📘 IHardwareController
- **책임**: 하드웨어 장치 기본 제어
- **행동**:
  - `+ initialize() -> bool`
  - `+ connect() -> bool`
  - `+ disconnect() -> bool`
  - `+ get_status() -> dict`
  - `+ reset() -> bool`
- **협력**: HardwareManager

## 📘 IRobotDriver
- **책임**: 로봇 제어
- **행동**:
  - `+ move_joint(joint_id: int, position: float, velocity: float) -> bool`
  - `+ move_to_pose(pose: dict) -> bool`
  - `+ get_current_joint_states() -> List[JointPosition]`
  - `+ get_current_pose() -> dict`
  - `+ enable_robot() -> bool`
  - `+ disable_robot() -> bool`
  - `+ emergency_stop() -> bool`
- **협력**: RobotControllerManager
- **비고**: 다양한 로봇 유형에 대한 일반적인 인터페이스

## 📘 ICameraDriver
- **책임**: 카메라 제어
- **행동**:
  - `+ start_streaming() -> bool`
  - `+ stop_streaming() -> bool`
  - `+ capture_image() -> numpy.ndarray`
  - `+ set_parameters(params: dict) -> bool`
  - `+ get_parameters() -> dict`
- **협력**: SensorManager
- **비고**: 다양한 카메라 유형 지원

## 📘 ISensorDriver
- **책임**: 센서 제어
- **행동**:
  - `+ read_sensor() -> SensorReading`
  - `+ calibrate() -> bool`
  - `+ set_parameters(params: dict) -> bool`
- **협력**: SensorManager
- **비고**: 다양한 센서 유형 통합

## 📘 IIOController
- **책임**: I/O 포트 제어
- **행동**:
  - `+ set_digital_output(port: int, state: bool) -> bool`
  - `+ get_digital_input(port: int) -> bool`
  - `+ set_analog_output(port: int, value: float) -> bool`
  - `+ get_analog_input(port: int) -> float`
- **협력**: IOManager
- **비고**: 디지털/아날로그 I/O 통합

## 📘 IStatePublisher
- **책임**: 하드웨어 상태 발행
- **행동**:
  - `+ publish_state(state: dict) -> None`
  - `+ publish_error(error: dict) -> None`
- **협력**: HardwareStatePublisher
- **비고**: 상태 정보 표준화 발행

## 📘 IHeartbeatGenerator
- **책임**: 하트비트 신호 생성
- **행동**:
  - `+ generate_heartbeat() -> HeartbeatSignal`
  - `+ start_heartbeat(interval_ms: int) -> bool`
  - `+ stop_heartbeat() -> bool`
  - `+ is_active() -> bool`
- **협력**: HeartbeatManager
- **비고**: 구성 요소 생존 여부 신호 생성

---

# ✅ 3. Core Class 상세 설명

## 📘 HardwareNode
- **책임**: ROS2 노드로 하드웨어 관리
- **속성**:
  - `- hardware_manager: HardwareManager`
  - `- state_publisher: HardwareStatePublisher`
  - `- command_subscriber: HardwareCommandSubscriber`
  - `- driver_factory: DriverFactory`
  - `- heartbeat_manager: HeartbeatManager`
- **행동**:
  - `+ initialize_hardware() -> None`
  - `+ handle_command(cmd: RobotCommand) -> None`
  - `+ publish_hardware_state() -> None`
  - `+ publish_heartbeats() -> None`
  - `+ cleanup_hardware() -> None`
  - `+ load_configuration() -> HardwareConfiguration`
- **비고**: ROS2 인터페이스 관리 및 하드웨어 초기화

## 📘 HardwareManager
- **책임**: 전체 하드웨어 리소스 관리
- **속성**:
  - `- robot_manager: RobotControllerManager`
  - `- sensor_manager: SensorManager`
  - `- io_manager: IOManager`
  - `- hardware_state: HardwareState`
- **행동**:
  - `+ initialize_all_systems() -> bool`
  - `+ shutdown_all_systems() -> bool`
  - `+ get_hardware_state() -> HardwareState`
  - `+ handle_hardware_error(error: dict) -> None`
- **비고**: 하드웨어 서브시스템 조율

## 📘 RobotControllerManager
- **책임**: 로봇 드라이버 관리
- **속성**:
  - `- robot_drivers: Dict[str, IRobotDriver]`
  - `- active_robot: str`
- **행동**:
  - `+ register_robot(name: str, driver: IRobotDriver) -> None`
  - `+ select_active_robot(name: str) -> bool`
  - `+ execute_command(cmd: RobotCommand) -> bool`
  - `+ get_all_robot_states() -> dict`
- **비고**: 다중 로봇 관리 지원

## 📘 SensorManager
- **책임**: 센서 드라이버 관리
- **속성**:
  - `- camera_drivers: Dict[str, ICameraDriver]`
  - `- sensor_drivers: Dict[str, ISensorDriver]`
- **행동**:
  - `+ register_camera(name: str, driver: ICameraDriver) -> None`
  - `+ register_sensor(name: str, driver: ISensorDriver) -> None`
  - `+ get_camera_image(name: str) -> numpy.ndarray`
  - `+ get_sensor_reading(name: str) -> SensorReading`
- **비고**: 다양한 유형의 센서 통합

## 📘 IOManager
- **책임**: I/O 포트 관리
- **속성**:
  - `- io_controllers: Dict[str, IIOController]`
  - `- io_state: IOState`
- **행동**:
  - `+ register_controller(name: str, controller: IIOController) -> None`
  - `+ set_output(controller: str, port: int, value: any) -> bool`
  - `+ get_input(controller: str, port: int) -> any`
  - `+ get_io_state() -> IOState`
- **비고**: 다양한 I/O 장치 관리

## 📘 HardwareStatePublisher
- **책임**: 하드웨어 상태 발행
- **속성**:
  - `- publishers: Dict[str, rclpy.Publisher]`
- **행동**:
  - `+ create_publisher(topic: str, msg_type: Type) -> None`
  - `+ publish_robot_state(state: dict) -> None`
  - `+ publish_sensor_data(data: dict) -> None`
  - `+ publish_io_state(state: IOState) -> None`
- **비고**: 다양한 하드웨어 상태 정보 발행

## 📘 HardwareCommandSubscriber
- **책임**: 하드웨어 명령 구독
- **속성**:
  - `- subscribers: Dict[str, rclpy.Subscription]`
  - `- command_handlers: Dict[str, Callable]`
- **행동**:
  - `+ create_subscription(topic: str, msg_type: Type, handler: Callable) -> None`
  - `+ process_command(msg: any) -> None`
- **비고**: 다양한 하드웨어 명령 수신 및 처리

## 📘 DriverFactory
- **책임**: 하드웨어 드라이버 생성
- **속성**:
  - `- driver_registry: Dict[str, Type]`
- **행동**:
  - `+ register_driver_type(name: str, driver_class: Type) -> None`
  - `+ create_driver(driver_type: str, params: dict) -> IHardwareController`
- **비고**: 하드웨어 추상화 및 생성 관리

## 📘 HeartbeatManager
- **책임**: 하트비트 관리 및 발행
- **속성**:
  - `- component_heartbeats: Dict[str, IHeartbeatGenerator]`
  - `- heartbeat_publishers: Dict[str, rclpy.Publisher]`
  - `- heartbeat_intervals: Dict[str, int]`
  - `- active_timers: Dict[str, rclpy.Timer]`
- **행동**:
  - `+ register_component(component_id: str, generator: IHeartbeatGenerator, interval_ms: int) -> None`
  - `+ start_all_heartbeats() -> bool`
  - `+ stop_all_heartbeats() -> bool`
  - `+ publish_heartbeat(component_id: str) -> None`
  - `+ get_all_heartbeat_statuses() -> Dict[str, bool]`
- **비고**: 시스템 모든 구성 요소의 하트비트 관리 및 발행

---

# ✅ 4. 데이터 클래스 상세 설명

## 📘 HardwareState
- **책임**: 전체 하드웨어 상태 표현
- **속성**:
  - `+ robot_states: Dict[str, dict]`
  - `+ sensor_states: Dict[str, dict]`
  - `+ io_state: IOState`
  - `+ error_flags: Dict[str, bool]`
  - `+ timestamp: float`
- **비고**: 하드웨어 요소의 종합적 상태 관리

## 📘 JointPosition
- **책임**: 로봇 관절 위치 표현
- **속성**:
  - `+ joint_id: int`
  - `+ position: float`
  - `+ velocity: float`
  - `+ effort: float`
  - `+ is_moving: bool`
- **비고**: 로봇 관절 상태 정보

## 📘 SensorReading
- **책임**: 센서 데이터 표현
- **속성**:
  - `+ value: any`
  - `+ timestamp: float`
  - `+ sensor_id: str`
  - `+ status: str`
- **비고**: 다양한 센서 데이터 표준화

## 📘 IOState
- **책임**: I/O 포트 상태 표현
- **속성**:
  - `+ digital_inputs: Dict[int, bool]`
  - `+ digital_outputs: Dict[int, bool]`
  - `+ analog_inputs: Dict[int, float]`
  - `+ analog_outputs: Dict[int, float]`
- **비고**: I/O 포트 상태 추적

## 📘 RobotCommand
- **책임**: 로봇 명령 표현
- **속성**:
  - `+ command_type: str`
  - `+ robot_id: str`
  - `+ parameters: dict`
  - `+ priority: int`
  - `+ timestamp: float`
- **비고**: 로봇 제어 명령 구조화

## 📘 HardwareConfiguration
- **책임**: 하드웨어 구성 표현
- **속성**:
  - `+ robot_configs: Dict[str, dict]`
  - `+ camera_configs: Dict[str, dict]`
  - `+ sensor_configs: Dict[str, dict]`
  - `+ io_configs: Dict[str, dict]`
  - `+ heartbeat_configs: Dict[str, dict]`
- **비고**: 하드웨어 초기화 및 구성 관리

## 📘 HeartbeatSignal
- **책임**: 하트비트 신호 데이터 표현
- **속성**:
  - `+ component_id: str`
  - `+ timestamp: float`
  - `+ status: str`
  - `+ details: dict`
- **비고**: 구성 요소의 상태 및 생존 정보 포함

---

# ✅ 5. 클래스 간 관계 요약 (UML 관계)

- HardwareNode → (Aggregation) → HardwareManager, HardwareStatePublisher, HardwareCommandSubscriber, DriverFactory, HeartbeatManager
- HardwareManager → (Aggregation) → RobotControllerManager, SensorManager, IOManager, HardwareState
- RobotControllerManager → (Association) → IRobotDriver
- SensorManager → (Association) → ICameraDriver, ISensorDriver
- IOManager → (Association) → IIOController
- HardwareStatePublisher → (Implements) → IStatePublisher
- DriverFactory → (Creates) → IHardwareController 및 파생 인터페이스
- HeartbeatManager → (Association) → IHeartbeatGenerator

---

# ✅ 6. ROS2 및 시스템 특성 반영

- HardwareNode는 다음 서비스/토픽 제공:
  - `/hardware/robot_command`
  - `/hardware/robot_state`
  - `/hardware/sensor_data`
  - `/hardware/io_control`
  - `/hardware/io_state`
  - `/hardware/error_status`
  - `/hardware/heartbeat/{component_id}`
- 하드웨어 상태 및 오류 발행
- 동기식/비동기식 명령 처리
- 다양한 센서 데이터 통합
- 로봇 및 I/O 제어 통합
- 주기적 하트비트 신호 발행

---

# ✅ 7. 예외/에러 상황 대응

| 상황 | 대응 방법 |
|:--|:--|
| 하드웨어 연결 실패 | 자동 재시도 및 오류 보고 |
| 로봇 통신 오류 | 안전한 상태로 전환 및 복구 시도 |
| 센서 데이터 이상 | 필터링 및 보정, 심각한 경우 알림 |
| 명령 실행 실패 | 오류 로깅 및 클라이언트에 피드백 |
| 드라이버 로드 실패 | 대체 드라이버 시도 및 상세 오류 보고 |
| 시스템 재부팅 | 안전한 상태 복원 절차 |
| 하트비트 발행 실패 | 구성 요소 재초기화 및 오류 로깅 |

---

# 📢 최종 요약

- 유연한 하드웨어 추상화 구조 설계
- 다양한 하드웨어 장치 통합을 위한 인터페이스 제공
- 구체적인 드라이버 구현 없이 인터페이스 및 관리 클래스 정의
- 향후 확장 가능한 모듈식 아키텍처
- 하드웨어 상태 모니터링 및 명령 처리 통합
- 오류 상황 관리 및 보고 매커니즘
- 다양한 로봇, 센서, I/O 장치의 통합 지원
- 모든 주요 구성 요소에 대한 하트비트 신호 생성 및 발행
- UseCase 요구사항에 맞춘 구조 제공

---
