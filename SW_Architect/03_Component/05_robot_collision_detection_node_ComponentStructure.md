# robot_collision_detection 패키지 – Component Structure

---

## 1. 컴포넌트 목록

### CollisionDetectionNodeComponent
- CollisionDetectionNode
- 책임: 충돌 감지 시스템 전체 조정 및 ROS2 인터페이스 제공

### CollisionDetectionComponent
- CollisionDetectionManager
- ICollisionDetector 구현체들
- 책임: 충돌 감지 알고리즘 관리 및 실행

### EnvironmentManagementComponent
- EnvironmentManager
- IEnvironmentRepresentation 구현체들
- 책임: 환경 정보 수집 및 관리

### CollisionResponseComponent
- CollisionResponseManager
- ICollisionResponseHandler 구현체들
- 책임: 충돌 감지 시 대응 전략 실행

### ConfigurationComponent
- CollisionDetectionConfig
- 책임: 시스템 설정 관리 및 업데이트

---

## 2. 컴포넌트 상세 설명

### CollisionDetectionNodeComponent
- **구성 요소**: CollisionDetectionNode
- **역할**:
  - ROS2 노드로서 외부 통신 인터페이스 제공
  - 충돌 감지 시스템 전체 흐름 제어
  - 로봇 상태 및 환경 정보 업데이트 처리
- **입력**:
  - `+ run_detection_loop() -> None`
  - `+ handle_robot_state_update(state: RobotState) -> None`
  - `+ update_configuration(config: dict) -> bool`
- **출력**:
  - 충돌 감지 상태 토픽
  - 충돌 경고 토픽
- **협력**:
  - CollisionDetectionComponent
  - EnvironmentManagementComponent
  - CollisionResponseComponent
  - ConfigurationComponent

### CollisionDetectionComponent
- **구성 요소**: CollisionDetectionManager, ICollisionDetector 구현체들
- **역할**:
  - 충돌 감지 알고리즘 실행 및 관리
  - 다양한 감지 알고리즘 전환 지원
- **입력**:
  - `+ check_collision(robot_state: RobotState, environment: List[EnvironmentObject]) -> CollisionState`
  - `+ switch_detector(detector_id: str) -> bool`
- **출력**:
  - 충돌 상태 정보 (CollisionState)
- **협력**:
  - CollisionDetectionNodeComponent
  - EnvironmentManagementComponent

### EnvironmentManagementComponent
- **구성 요소**: EnvironmentManager, IEnvironmentRepresentation 구현체들
- **역할**:
  - 환경 정보 수집 및 표현
  - 물체 위치 및 형상 정보 관리
- **입력**:
  - `+ update_environment(data: dict) -> None`
  - `+ get_environment() -> List[EnvironmentObject]`
- **출력**:
  - 환경 물체 리스트
- **협력**:
  - CollisionDetectionNodeComponent
  - CollisionDetectionComponent

### CollisionResponseComponent
- **구성 요소**: CollisionResponseManager, ICollisionResponseHandler 구현체들
- **역할**:
  - 충돌 감지 시 적절한 대응 전략 실행
  - 상황에 맞는 대응 핸들러 선택
- **입력**:
  - `+ handle_collision(collision_state: CollisionState) -> bool`
  - `+ switch_handler(handler_id: str) -> bool`
- **출력**:
  - 대응 실행 결과
  - 대응 명령 (외부 시스템에)
- **협력**:
  - CollisionDetectionNodeComponent
  - 외부 제어 시스템 (명령 전송용)

### ConfigurationComponent
- **구성 요소**: CollisionDetectionConfig
- **역할**:
  - 시스템 설정 로드 및 관리
  - 설정 변경 요청 처리
- **입력**:
  - `+ load_config(file_path: str) -> bool`
  - `+ update_config(config: dict) -> bool`
  - `+ get_config_section(section: str) -> dict`
- **출력**:
  - 설정 정보
- **협력**:
  - 모든 컴포넌트

---

## 3. 컴포넌트 간 관계 요약

- CollisionDetectionNodeComponent → CollisionDetectionComponent (충돌 감지 요청)
- CollisionDetectionNodeComponent → EnvironmentManagementComponent (환경 정보 요청)
- CollisionDetectionNodeComponent → CollisionResponseComponent (대응 요청)
- CollisionDetectionNodeComponent → ConfigurationComponent (설정 요청)
- CollisionDetectionComponent → EnvironmentManagementComponent (환경 정보 사용)
- CollisionDetectionComponent → ConfigurationComponent (감지 설정 사용)
- CollisionResponseComponent → ConfigurationComponent (대응 설정 사용)

---

## 4. 요약

| 컴포넌트 | 책임 | 협력 |
|:--|:--|:--|
| CollisionDetectionNodeComponent | 전체 시스템 조정 및 ROS2 인터페이스 | 모든 컴포넌트 |
| CollisionDetectionComponent | 충돌 감지 알고리즘 실행 | Node, Environment |
| EnvironmentManagementComponent | 환경 정보 수집 및 관리 | Node, Detection |
| CollisionResponseComponent | 충돌 대응 전략 실행 | Node, 외부 시스템 |
| ConfigurationComponent | 설정 관리 | 모든 컴포넌트 |

---

## 5. 최종 정리

- 인터페이스 기반의 모듈화된 시스템 아키텍처
- 감지, 환경 표현, 대응을 분리한 책임 명확한 구조
- 플러그인 방식의 알고리즘 교체 가능성
- 실시간 설정 변경 지원
- 안전 우선의 보수적인 충돌 감지 및 대응 전략
- ROS2 기반의 유연한 통합 인터페이스

---