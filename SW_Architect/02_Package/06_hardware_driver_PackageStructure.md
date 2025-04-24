## 🧩 h_w_node 패키지 고급 설계 문서 (최종 수정본)

---

### 1. 📌 책임 (Responsibility)

`h_w_node` 패키지는 로봇을 구성하는 모든 실제 장치들과의 통신을 담당하는 하위 계층이다.  
각 장치는 하나의 공통 인터페이스를 따르며, 이 패키지는 다음과 같은 역할을 수행한다:

- 로봇팔, 카메라, 허리, 도어, 스피커 등의 장치 드라이버 제어
- 상위 명령(`command_executor`)으로부터 요청된 명령을 실제 장치 명령으로 변환
- 각 장치의 상태를 주기적으로 읽어 publish
- 센서 또는 장치 이벤트를 다른 패키지에 전달
- 모든 주요 구성 요소의 하트비트 신호 생성 및 발행
- 다양한 하드웨어 장치의 통합 인터페이스 제공

---

### 2. 🔗 패키지 의존 관계 (Dependencies)

| 연결 대상 패키지 | 연결 방식 | 설명 |
|------------------|-----------|------|
| `command_executor` | ROS2 Service | 동작 실행 요청 수신 (pick, place, door 등) |
| `status_reporter` | ROS2 Topic Publish | 위치, 배터리, 장치 상태 정보 전달 |
| `vision_node` | 카메라 스트림 제공 | 실시간 영상 이미지 전송 |
| `safety_interlock_node` | ROS2 Topic Publish/Subscribe | 센서 정보 전달 및 안전 명령 수신 |
| `robot_collision_detection` | ROS2 Topic Publish | 센서 데이터 및 로봇 상태 제공 |

---

### 3. 📬 외부 인터페이스 정의 (Interfaces)

#### ✅ 제공하는 ROS2 토픽

| 토픽명 | 타입 | 설명 |
|--------|------|------|
| `/hardware/robot_state` | `custom_msgs/msg/RobotState` | 로봇 상태 정보 |
| `/hardware/sensor_data` | `custom_msgs/msg/SensorData` | 센서 데이터 |
| `/hardware/io_state` | `custom_msgs/msg/IOState` | I/O 포트 상태 |
| `/hardware/error_status` | `custom_msgs/msg/ErrorStatus` | 오류 상태 |
| `/hardware/camera/image_raw` | `sensor_msgs/msg/Image` | 실시간 이미지 |
| `/hardware/emergency_button` | `std_msgs/msg/Bool` | 비상 버튼 상태 |
| `/hardware/heartbeat/{component_id}` | `custom_msgs/msg/HeartbeatSignal` | 구성 요소 하트비트 신호 |

#### ✅ 제공하는 ROS2 서비스

| 서비스명 | 타입 | 설명 |
|----------|------|------|
| `/hardware/robot_command` | `custom_interfaces/srv/RobotCommand` | 로봇 명령 실행 |
| `/hardware/io_control` | `custom_interfaces/srv/IOControl` | I/O 제어 |
| `/hardware/camera_control` | `custom_interfaces/srv/CameraControl` | 카메라 제어 |

---

### 4. ⚙️ 내부 구성 요소 (컴포넌트 구조)

#### 인터페이스 (추상 클래스)
- `IHardwareController` (하드웨어 제어 인터페이스)
- `IRobotDriver` (로봇 드라이버 인터페이스)
- `ICameraDriver` (카메라 드라이버 인터페이스)
- `ISensorDriver` (센서 드라이버 인터페이스)
- `IIOController` (I/O 제어 인터페이스)
- `IStatePublisher` (상태 발행 인터페이스)
- `IHeartbeatGenerator` (하트비트 생성 인터페이스)

#### 핵심 클래스
| 컴포넌트명 | 타입 | 역할 |
|------------|------|------|
| `HardwareNode` | ROS2 Node | 전체 하드웨어 관리 및 ROS2 인터페이스 제공 |
| `HardwareManager` | Class | 하드웨어 리소스 통합 관리 |
| `RobotControllerManager` | Class | 로봇 드라이버 관리 |
| `SensorManager` | Class | 센서 드라이버 관리 |
| `IOManager` | Class | I/O 포트 관리 |
| `HardwareStatePublisher` | Class | 하드웨어 상태 발행 |
| `HardwareCommandSubscriber` | Class | 하드웨어 명령 구독 |
| `DriverFactory` | Class | 하드웨어 드라이버 생성 |
| `HeartbeatManager` | Class | 하트비트 관리 및 발행 |

#### 데이터 클래스
- `HardwareState` (하드웨어 상태 데이터)
- `JointPosition` (관절 위치 데이터)
- `SensorReading` (센서 읽기 데이터)
- `IOState` (I/O 상태 데이터)
- `RobotCommand` (로봇 명령 데이터)
- `HardwareConfiguration` (하드웨어 구성 데이터)
- `HeartbeatSignal` (하트비트 신호 데이터)

---

### 5. 🔄 동작 흐름 예시

#### 로봇 명령 실행 흐름
1. `command_executor`가 `/hardware/robot_command` 서비스 호출
2. `HardwareNode`가 명령을 수신하여 `HardwareCommandSubscriber`로 전달
3. `HardwareCommandSubscriber`가 명령을 `RobotCommand` 객체로 변환
4. `HardwareManager`가 명령을 `RobotControllerManager`로 전달
5. `RobotControllerManager`가 적절한 `IRobotDriver` 구현체를 선택하여 명령 실행
6. 명령 실행 결과를 서비스 응답으로 반환
7. `HardwareStatePublisher`가 로봇 상태를 토픽으로 발행

#### 하트비트 생성 흐름
1. `HeartbeatManager`가 시스템 시작 시 `start_all_heartbeats()` 호출
2. 각 구성 요소 별로 등록된 `IHeartbeatGenerator` 구현체가 하트비트 생성
3. 지정된 주기로 `HeartbeatSignal` 데이터 생성
4. `/hardware/heartbeat/{component_id}` 토픽으로 하트비트 신호 발행
5. `safety_interlock_node`가 하트비트 신호를 수신하여 구성 요소 상태 모니터링

---

### 6. 📊 클래스 간 관계 및 상호작용

- HardwareNode는 HardwareManager, HardwareStatePublisher, HardwareCommandSubscriber, DriverFactory, HeartbeatManager를 포함(Aggregation)
- HardwareManager는 RobotControllerManager, SensorManager, IOManager를 포함
- RobotControllerManager는 여러 IRobotDriver 구현체들과 연결(Association)
- SensorManager는 ICameraDriver, ISensorDriver 구현체들과 연결
- IOManager는 IIOController 구현체들과 연결
- HeartbeatManager는 여러 IHeartbeatGenerator 구현체들과 연결

---

### 7. ⚠️ 예외/에러 상황 대응

| 상황 | 대응 방법 |
|------|----------|
| 하드웨어 연결 실패 | 자동 재시도 및 오류 보고 |
| 로봇 통신 오류 | 안전한 상태로 전환 및 복구 시도 |
| 센서 데이터 이상 | 필터링 및 보정, 심각한 경우 알림 |
| 명령 실행 실패 | 오류 로깅 및 클라이언트에 피드백 |
| 드라이버 로드 실패 | 대체 드라이버 시도 및 상세 오류 보고 |
| 시스템 재부팅 | 안전한 상태 복원 절차 |
| 하트비트 발행 실패 | 구성 요소 재초기화 및 오류 로깅 |

---

### 8. 📢 최종 요약

- 인터페이스 기반 설계로 유연한 하드웨어 추상화 구조 제공
- 다양한 하드웨어 장치 통합을 위한 공통 인터페이스 구현
- 로봇, 센서, I/O 장치의 통합 관리 메커니즘
- 상태 모니터링 및 명령 처리 통합 구조
- 모든 주요 구성 요소의 하트비트 신호 생성 및 발행
- 오류 상황 관리 및 보고 체계 구축
- 향후 확장 가능한 모듈식 아키텍처 설계