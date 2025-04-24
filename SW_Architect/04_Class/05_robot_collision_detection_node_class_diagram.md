# 🧩 robot_collision_detection 패키지 – Class Diagram (수정 가능한 기본 구조)

---

# ✅ 1. 클래스 목록 및 구조

### 📘 Interface (추상 클래스)
- `ICollisionDetector` (충돌 감지 인터페이스)
- `ICollisionResponseHandler` (충돌 대응 인터페이스)
- `IEnvironmentRepresentation` (환경 표현 인터페이스)

### 📘 Core Classes
- `CollisionDetectionNode` (rclpy.Node, 충돌 감지 노드)
- `CollisionDetectionManager` (충돌 감지 관리자)
- `EnvironmentManager` (환경 관리)
- `CollisionResponseManager` (충돌 대응 관리)
- `CollisionDetectionConfig` (설정 관리)

### 📘 데이터 구조체
- `CollisionState` (충돌 상태 데이터)
- `EnvironmentObject` (환경 물체 데이터)
- `RobotState` (로봇 상태 데이터)

---

# ✅ 2. 인터페이스 상세 설명

## 📘 ICollisionDetector
- **책임**: 충돌 감지 알고리즘 구현
- **행동**:
  - `+ detect_collision(robot_state: RobotState, environment: List[EnvironmentObject]) -> CollisionState`
  - `+ update_parameters(params: dict) -> None`
- **협력**: CollisionDetectionManager
- **비고**: 다양한 충돌 감지 알고리즘 플러그인 가능

## 📘 ICollisionResponseHandler
- **책임**: 충돌 감지 시 대응 구현
- **행동**:
  - `+ handle_collision(collision_state: CollisionState) -> bool`
  - `+ reset() -> None`
- **협력**: CollisionResponseManager
- **비고**: 다양한 대응 전략 구현 가능

## 📘 IEnvironmentRepresentation
- **책임**: 환경 표현 방식 구현
- **행동**:
  - `+ update_environment(data: dict) -> None`
  - `+ get_objects() -> List[EnvironmentObject]`
- **협력**: EnvironmentManager
- **비고**: 다양한 환경 표현 방식 지원

---

# ✅ 3. Core Class 상세 설명

## 📘 CollisionDetectionNode
- **책임**: ROS2 인터페이스 및 충돌 감지 시스템 관리
- **속성**:
  - `- detection_manager: CollisionDetectionManager`
  - `- environment_manager: EnvironmentManager`
  - `- response_manager: CollisionResponseManager`
  - `- config: CollisionDetectionConfig`
- **행동**:
  - `+ run_detection_loop() -> None`
  - `+ handle_robot_state_update(state: RobotState) -> None`
  - `+ update_configuration(config: dict) -> bool`
- **협력**: 다른 Core 클래스들, ROS2 Topic/Service
- **비고**: ROS2 Node로 동작, 시스템 전체 관리

## 📘 CollisionDetectionManager
- **책임**: 충돌 감지 알고리즘 관리 및 실행
- **속성**:
  - `- detectors: List[ICollisionDetector]`
  - `- active_detector: ICollisionDetector`
- **행동**:
  - `+ check_collision(robot_state: RobotState, environment: List[EnvironmentObject]) -> CollisionState`
  - `+ switch_detector(detector_id: str) -> bool`
- **협력**: ICollisionDetector 구현체
- **비고**: 런타임에 충돌 감지 알고리즘 교체 가능

## 📘 EnvironmentManager
- **책임**: 환경 정보 관리
- **속성**:
  - `- environment_representation: IEnvironmentRepresentation`
  - `- environment_objects: List[EnvironmentObject]`
- **행동**:
  - `+ update_environment(data: dict) -> None`
  - `+ get_environment() -> List[EnvironmentObject]`
- **협력**: IEnvironmentRepresentation 구현체
- **비고**: 환경 정보 업데이트 및 제공

## 📘 CollisionResponseManager
- **책임**: 충돌 감지 시 대응 관리
- **속성**:
  - `- response_handlers: List[ICollisionResponseHandler]`
  - `- active_handler: ICollisionResponseHandler`
- **행동**:
  - `+ handle_collision(collision_state: CollisionState) -> bool`
  - `+ switch_handler(handler_id: str) -> bool`
- **협력**: ICollisionResponseHandler 구현체
- **비고**: 충돌 상황에 따른 적절한 대응 전략 선택

## 📘 CollisionDetectionConfig
- **책임**: 충돌 감지 설정 관리
- **속성**:
  - `- detection_params: dict`
  - `- response_params: dict`
  - `- environment_params: dict`
- **행동**:
  - `+ load_config(file_path: str) -> bool`
  - `+ update_config(config: dict) -> bool`
  - `+ get_config_section(section: str) -> dict`
- **협력**: 모든 Core 클래스
- **비고**: 설정 파일 로드 및 동적 설정 변경 지원

---

# ✅ 4. 데이터 구조체 상세 설명

## 📘 CollisionState
- **속성**:
  - `+ is_collision: bool`
  - `+ collision_points: List[dict]`
  - `+ collision_objects: List[EnvironmentObject]`
  - `+ severity: float`
  - `+ time_to_collision: float`

## 📘 EnvironmentObject
- **속성**:
  - `+ id: str`
  - `+ type: str`
  - `+ position: List[float]`
  - `+ orientation: List[float]`
  - `+ geometry: dict`
  - `+ is_static: bool`

## 📘 RobotState
- **속성**:
  - `+ joint_positions: List[float]`
  - `+ joint_velocities: List[float]`
  - `+ end_effector_pose: dict`
  - `+ planned_trajectory: List[dict]`

---

# ✅ 5. 클래스 간 관계 요약 (UML 관계)

- CollisionDetectionNode → (Aggregation) → CollisionDetectionManager, EnvironmentManager, CollisionResponseManager, CollisionDetectionConfig
- CollisionDetectionManager → (Association) → ICollisionDetector
- EnvironmentManager → (Association) → IEnvironmentRepresentation
- CollisionResponseManager → (Association) → ICollisionResponseHandler

---

# ✅ 6. ROS2 및 시스템 특성 반영

- CollisionDetectionNode는 다음 토픽 구독:
  - `/robot_state` : 로봇 상태 정보
  - `/environment_updates` : 환경 정보 업데이트
- CollisionDetectionNode는 다음 토픽 발행:
  - `/collision_detection/state` : 충돌 감지 상태
  - `/collision_detection/warnings` : 충돌 경고
- CollisionDetectionNode는 다음 서비스 제공:
  - `/collision_detection/update_config` : 설정 업데이트
  - `/collision_detection/switch_detector` : 감지기 변경
  - `/collision_detection/switch_response` : 대응 전략 변경

---

# ✅ 7. 유연성 및 확장성

- **플러그인 아키텍처**: 인터페이스 기반 설계로 새로운 감지/대응 알고리즘 쉽게 추가 가능
- **설정 가능성**: 모든 파라미터가 동적으로 조정 가능한 구조
- **모듈화**: 환경 표현, 충돌 감지, 대응 전략이 모두 분리되어 독립적 개발 가능
- **런타임 교체**: 실행 중에도 알고리즘 교체 가능한 구조

---

# ✅ 8. 예외/에러 상황 대응

| 상황 | 대응 방법 |
|:--|:--|
| 감지기 초기화 실패 | 대체 감지기로 자동 전환 |
| 환경 데이터 불완전 | 보수적 판단으로 안전성 확보 |
| 충돌 대응 실패 | 기본 비상 정지 절차 수행 |
| 설정 로드 실패 | 기본값으로 초기화 |
| 로봇 상태 데이터 지연 | 예측 모델로 임시 보완 |

---

# 📢 최종 요약

- 인터페이스 기반의 유연한 구조
- 다양한 알고리즘 지원 가능한 플러그인 아키텍처
- 안전 우선의 보수적 설계
- 실행 중 변경 가능한 구성 요소
- 향후 요구사항 변화에 쉽게 대응 가능한 확장성

---
