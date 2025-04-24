# h_w_node 패키지 – Component Structure (최종 수정본)

---

## 1. 컴포넌트 목록

### HardwareNodeComponent
- HardwareNode
- 책임: 하드웨어 통합 제어 및 ROS2 서비스/토픽 관리

### HardwareManagementComponent
- HardwareManager
- DriverFactory
- 책임: 하드웨어 리소스 관리 및 드라이버 생성

### RobotControlComponent
- RobotControllerManager
- IRobotDriver 구현체들
- 책임: 로봇 드라이버 관리 및 명령 실행

### SensorControlComponent
- SensorManager
- ICameraDriver 구현체들
- ISensorDriver 구현체들
- 책임: 센서 및 카메라 관리

### IOControlComponent
- IOManager
- IIOController 구현체들
- 책임: 입출력 포트 관리

### StatePublishingComponent
- HardwareStatePublisher
- 책임: 하드웨어 상태 발행

### CommandInterfaceComponent
- HardwareCommandSubscriber
- 책임: 하드웨어 명령 구독 및 처리

### HeartbeatComponent
- HeartbeatManager
- IHeartbeatGenerator 구현체들
- 책임: 하트비트 생성 및 관리

---

## 2. 컴포넌트 상세 설명

### HardwareNodeComponent
- **구성 요소**: HardwareNode
- **역할**:
  - ROS2 노드로서 하드웨어 관리
  - 서비스/토픽 등록 및 요청 처리
  - 전체 하드웨어 컴포넌트 조정
- **입력**:
  - `+ initialize_hardware() -> None`
  - `+ handle_command(cmd: RobotCommand) -> None`
- **출력**:
  - 하드웨어 상태 및 하트비트 발행
- **협력**:
  - 모든 하드웨어 컴포넌트

### HardwareManagementComponent
- **구성 요소**: HardwareManager, DriverFactory
- **역할**:
  - 전체 하드웨어 리소스 관리
  - 다양한 하드웨어 드라이버 생성 및 등록
- **입력**:
  - `+ initialize_all_systems() -> bool`
  - `+ register_driver_type(name: str, driver_class: Type) -> None`
  - `+ create_driver(driver_type: str, params: dict) -> IHardwareController`
- **출력**:
  - 초기화된 드라이버 인스턴스
  - 하드웨어 상태 정보
- **협력**:
  - HardwareNodeComponent
  - 다른 모든 컨트롤 컴포넌트

### RobotControlComponent
- **구성 요소**: RobotControllerManager, IRobotDriver 구현체들
- **역할**:
  - 로봇 드라이버 관리
  - 로봇 제어 명령 실행
- **입력**:
  - `+ register_robot(name: str, driver: IRobotDriver) -> None`
  - `+ execute_command(cmd: RobotCommand) -> bool`
- **출력**:
  - 명령 실행 결과
  - 로봇 상태 정보
- **협력**:
  - HardwareManagementComponent
  - StatePublishingComponent

### SensorControlComponent
- **구성 요소**: SensorManager, ICameraDriver 및 ISensorDriver 구현체들
- **역할**:
  - 센서 및 카메라 드라이버 관리
  - 데이터 수집 및 설정 제어
- **입력**:
  - `+ register_camera(name: str, driver: ICameraDriver) -> None`
  - `+ register_sensor(name: str, driver: ISensorDriver) -> None`
  - `+ get_camera_image(name: str) -> numpy.ndarray`
  - `+ get_sensor_reading(name: str) -> SensorReading`
- **출력**:
  - 센서 데이터
  - 카메라 이미지
- **협력**:
  - HardwareManagementComponent
  - StatePublishingComponent

### IOControlComponent
- **구성 요소**: IOManager, IIOController 구현체들
- **역할**:
  - 입출력 포트 관리
  - 디지털/아날로그 입출력 제어
- **입력**:
  - `+ register_controller(name: str, controller: IIOController) -> None`
  - `+ set_output(controller: str, port: int, value: any) -> bool`
  - `+ get_input(controller: str, port: int) -> any`
- **출력**:
  - 입출력 설정 결과
  - 입력 데이터
- **협력**:
  - HardwareManagementComponent
  - StatePublishingComponent

### StatePublishingComponent
- **구성 요소**: HardwareStatePublisher
- **역할**:
  - 하드웨어 상태 정보 발행
  - 에러 상태 발행
- **입력**:
  - `+ publish_robot_state(state: dict) -> None`
  - `+ publish_sensor_data(data: dict) -> None`
  - `+ publish_io_state(state: IOState) -> None`
- **출력**:
  - ROS2 토픽 메시지
- **협력**:
  - HardwareNodeComponent
  - RobotControlComponent
  - SensorControlComponent
  - IOControlComponent

### CommandInterfaceComponent
- **구성 요소**: HardwareCommandSubscriber
- **역할**:
  - 하드웨어 명령 구독
  - 명령 처리 및 라우팅
- **입력**:
  - `+ create_subscription(topic: str, msg_type: Type, handler: Callable) -> None`
  - ROS2 토픽 메시지
- **출력**:
  - 명령 처리 결과
- **협력**:
  - HardwareNodeComponent
  - HardwareManagementComponent

### HeartbeatComponent
- **구성 요소**: HeartbeatManager, IHeartbeatGenerator 구현체들
- **역할**:
  - 하트비트 신호 생성 및 관리
  - 구성 요소 상태 모니터링
- **입력**:
  - `+ register_component(component_id: str, generator: IHeartbeatGenerator, interval_ms: int) -> None`
  - `+ start_all_heartbeats() -> bool`
- **출력**:
  - 하트비트 신호
- **협력**:
  - HardwareNodeComponent
  - 안전 인터락 노드

---

## 3. 컴포넌트 간 관계 요약

- HardwareNodeComponent → HardwareManagementComponent (리소스 관리 위임)
- HardwareNodeComponent → CommandInterfaceComponent (명령 구독 관리)
- HardwareNodeComponent → StatePublishingComponent (상태 발행 관리)
- HardwareNodeComponent → HeartbeatComponent (하트비트 발행 관리)
- HardwareManagementComponent → RobotControlComponent (로봇 관리 위임)
- HardwareManagementComponent → SensorControlComponent (센서 관리 위임)
- HardwareManagementComponent → IOControlComponent (I/O 관리 위임)
- RobotControlComponent → StatePublishingComponent (로봇 상태 제공)
- SensorControlComponent → StatePublishingComponent (센서 데이터 제공)
- IOControlComponent → StatePublishingComponent (I/O 상태 제공)

---

## 4. 요약

| 컴포넌트 | 책임 | 협력 |
|:--|:--|:--|
| HardwareNodeComponent | ROS2 인터페이스 및 전체 조정 | 모든 컴포넌트 |
| HardwareManagementComponent | 하드웨어 리소스 및 드라이버 관리 | 모든 컨트롤 컴포넌트 |
| RobotControlComponent | 로봇 드라이버 관리 및 명령 실행 | HardwareManagement, StatePublishing |
| SensorControlComponent | 센서/카메라 관리 및 데이터 수집 | HardwareManagement, StatePublishing |
| IOControlComponent | 입출력 포트 관리 | HardwareManagement, StatePublishing |
| StatePublishingComponent | 하드웨어 상태 발행 | 모든 데이터 생성 컴포넌트 |
| CommandInterfaceComponent | 하드웨어 명령 구독 및 처리 | HardwareNode, HardwareManagement |
| HeartbeatComponent | 하트비트 신호 생성 및 관리 | HardwareNode, 안전 인터락 노드 |

---

## 최종 정리

- 하드웨어 추상화 인터페이스를 통한 모듈화된 구조
- 다양한 하드웨어 통합을 위한 유연한 드라이버 시스템
- 상태 발행 및 명령 구독 분리를 통한 책임 명확화
- 하트비트 생성 기능을 통한 시스템 안전성 강화
- 확장 가능한 컴포넌트 기반 아키텍처

---
