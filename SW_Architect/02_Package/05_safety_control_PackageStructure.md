## 🧩 safety_interlock_node 패키지 고급 설계 문서 (최종 수정본)

---

### 1. 📌 책임 (Responsibility)

`safety_interlock_node` 패키지는 로봇의 안전성과 관련된 모든 상황(비상정지, 충돌 감지, 구성요소 이상 등)을 처리하는 보호 계층으로,  
실시간으로 위험 요소를 감지하고 시스템 전체의 동작을 안전하게 중단시키는 역할을 한다.

- 외부 센서로부터 비상정지 신호, 충돌 위험 데이터 수신
- 다양한 안전 조건 검사 (관절 제한, 속도 제한, 충돌 검사 등)
- 각 하드웨어 구성 요소의 하트비트 모니터링
- 허리축, 양쪽 협동로봇, 머리 모터, 카메라, AGV 등의 상태 감시
- 비상 상황 발생 시 적절한 대응 조치 실행
- 시스템 인터락 제어를 통한 안전 확보
- 안전 관련 알림 전파

---

### 2. 🔗 패키지 의존 관계 (Dependencies)

| 연결 대상 패키지 | 연결 방식 | 설명 |
|------------------|-----------|------|
| `command_executor` | ROS2 Topic Publish | 실행 중 동작 중단 명령 송신 |
| `task_manager` | ROS2 Topic Publish | 시나리오 흐름 중단 알림 |
| `status_reporter` | ROS2 Topic Publish | UI 또는 ACS로 위험 상황 전달 |
| `h_w_node` | ROS2 Topic Subscribe | 비상 버튼, 센서 데이터, 하트비트 신호 수신 |
| `robot_collision_detection` | ROS2 Topic Subscribe | 충돌 위험 정보 수신 |
| `ui_interface` | WebSocket | 비상정지 알림 및 상태 전송 |
| `acs_interface` | WebSocket | 비상 상황 알림 전송 |

---

### 3. 📬 외부 인터페이스 정의 (Interfaces)

#### ✅ 제공하는 ROS2 서비스

| 서비스명 | 타입 | 설명 |
|----------|------|------|
| `/safety/check_status` | `custom_interfaces/srv/CheckSafetyStatus` | 현재 안전 상태 확인 |
| `/safety/emergency_stop` | `std_srvs/srv/Trigger` | 비상 정지 명령 수행 |
| `/safety/reset_system` | `std_srvs/srv/Trigger` | 안전 시스템 리셋 |
| `/safety/interlock_control` | `custom_interfaces/srv/InterlockControl` | 인터락 제어 |

#### ✅ 발행하는 ROS2 토픽

| 토픽명 | 타입 | 설명 |
|--------|------|------|
| `/safety/status` | `custom_msgs/msg/SafetyStatus` | 현재 안전 상태 정보 |
| `/safety/violations` | `custom_msgs/msg/SafetyViolations` | 안전 규칙 위반 정보 |
| `/safety/emergency` | `custom_msgs/msg/EmergencyStatus` | 비상 상태 정보 |
| `/safety/heartbeat` | `custom_msgs/msg/HeartbeatStatus` | 구성 요소 하트비트 상태 |

#### ✅ 구독하는 ROS2 토픽

| 토픽명 | 타입 | 설명 |
|--------|------|------|
| `/hardware/emergency_button` | `std_msgs/msg/Bool` | 비상 정지 버튼 상태 |
| `/hardware/sensor_data` | `custom_msgs/msg/SensorData` | 센서 데이터 |
| `/hardware/heartbeat` | `custom_msgs/msg/ComponentHeartbeat` | 하드웨어 구성 요소 하트비트 |
| `/collision/risk` | `custom_msgs/msg/CollisionRisk` | 충돌 위험 정보 |

---

### 4. ⚙️ 내부 구성 요소 (컴포넌트 구조)

#### 인터페이스 (추상 클래스)
- `ISafetyCheck` (안전 검사 인터페이스)
- `IEmergencyHandler` (비상 상황 처리 인터페이스)
- `ISafetyNotifier` (안전 알림 인터페이스)
- `IInterlockController` (인터락 제어 인터페이스)
- `IHeartbeatMonitor` (하트비트 모니터링 인터페이스)

#### 핵심 클래스
| 컴포넌트명 | 타입 | 역할 |
|------------|------|------|
| `SafetyInterlockNode` | ROS2 Node | 안전 시스템 전체 관리 및 ROS2 서비스/토픽 제공 |
| `SafetyManager` | Class | 안전 검사 관리 |
| `EmergencyManager` | Class | 비상 상황 처리 관리 |
| `InterlockController` | Class | 물리적 인터락 장치 제어 |
| `SafetyMonitor` | Class | 안전 상태 실시간 모니터링 |
| `SafetyNotificationManager` | Class | 안전 관련 알림 관리 |
| `HeartbeatMonitorManager` | Class | 하트비트 모니터링 관리 |

#### 안전 검사 구현체
- `JointLimitCheck` (implements ISafetyCheck)
- `SpeedLimitCheck` (implements ISafetyCheck)
- `WorkspaceCheck` (implements ISafetyCheck)
- `CollisionCheck` (implements ISafetyCheck)
- `PowerSystemCheck` (implements ISafetyCheck)
- `ToolStateCheck` (implements ISafetyCheck)

#### 비상 처리 구현체
- `EmergencyStop` (implements IEmergencyHandler)
- `SafetyPowerCut` (implements IEmergencyHandler)
- `SafePoseRecovery` (implements IEmergencyHandler)
- `AlarmActivation` (implements IEmergencyHandler)

#### 하트비트 모니터 구현체
- `WaistHeartbeatMonitor` (implements IHeartbeatMonitor)
- `LeftArmHeartbeatMonitor` (implements IHeartbeatMonitor)
- `RightArmHeartbeatMonitor` (implements IHeartbeatMonitor)
- `HeadMotorHeartbeatMonitor` (implements IHeartbeatMonitor)
- `CameraHeartbeatMonitor` (implements IHeartbeatMonitor)
- `AGVHeartbeatMonitor` (implements IHeartbeatMonitor)

---

### 5. 🔄 동작 흐름 예시

#### 안전 검사 흐름
1. `SafetyInterlockNode`가 주기적으로 `run_safety_checks()` 호출
2. `SafetyManager`가 등록된 모든 `ISafetyCheck` 구현체의 검사 실행
3. 위반 사항 발견 시 `EmergencyManager`를 통해 대응 조치 실행
4. `SafetyNotificationManager`를 통해 관련 시스템에 알림 전파

#### 비상 정지 흐름
1. 비상 정지 버튼 신호 수신 또는 안전 검사 실패
2. `EmergencyManager`의 `trigger_emergency()` 메소드 호출
3. 등록된 모든 `IEmergencyHandler` 구현체의 처리 실행
4. `InterlockController`를 통해 하드웨어 인터락 활성화
5. 상태 정보 발행 및 알림 전파

#### 하트비트 모니터링 흐름
1. `HeartbeatMonitorManager`가 주기적으로 `check_all_heartbeats()` 호출
2. 각 `IHeartbeatMonitor` 구현체가 담당 구성 요소의 하트비트 확인
3. 하트비트 손실 감지 시 해당 구성 요소에 적절한 대응 실행
4. 다중 구성 요소 하트비트 손실 시 전체 시스템 인터락 활성화

---

### 6. 📊 클래스 간 관계 및 상호작용

- SafetyInterlockNode는 SafetyManager, EmergencyManager, InterlockController, HeartbeatMonitorManager를 포함(Aggregation)
- SafetyManager는 여러 ISafetyCheck 구현체들과 연결(Association)
- EmergencyManager는 여러 IEmergencyHandler 구현체들과 연결
- SafetyNotificationManager는 여러 ISafetyNotifier 구현체들과 연결
- HeartbeatMonitorManager는 여러 IHeartbeatMonitor 구현체들과 연결

---

### 7. ⚠️ 예외/에러 상황 대응

| 상황 | 대응 방법 |
|------|----------|
| 안전 검사 실패 | 즉시 비상 정지 |
| 하드웨어 인터락 실패 | 백업 시스템 활성화 |
| 통신 두절 | 자동 안전 모드 전환 |
| 센서 오류 | 보수적 안전 판단 |
| 전원 이상 | 안전한 종료 수행 |
| 허리축 하트비트 손실 | 허리 모터 전원 차단 및 인터락 활성화 |
| 협동로봇 하트비트 손실 | 해당 로봇 비상정지 및 인터락 활성화 |
| 머리 모터 하트비트 손실 | 머리 모터 전원 차단 |
| 카메라 하트비트 손실 | 경고 알림 및 로깅 |
| AGV 하트비트 손실 | AGV 비상정지 명령 송신 |
| 다중 구성요소 하트비트 손실 | 전체 시스템 인터락 활성화 |

---

### 8. 📢 최종 요약

- 인터페이스 기반 설계로 유연한 안전 시스템 구조 제공
- 다양한 안전 검사 및 비상 상황 대응 메커니즘 구현
- 모든 주요 구성 요소 하트비트 모니터링 통합
- 실시간 안전 상태 모니터링 및 즉각 대응
- 다중 안전 장치 지원으로 신뢰성 확보
- 안전한 복구 절차 구현
- 구성 요소별 이상 상태 감지 및 차별화된 대응
- 시스템 생존성 확보를 위한 선제적 안전 관리