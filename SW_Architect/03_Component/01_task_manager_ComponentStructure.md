# task_manager 패키지 – Component Structure (최종 수정본)

---

# 1. 컴포넌트 목록

### TaskManagerNodeComponent
- TaskManagerNode (ROS2 노드)
- 책임: 시나리오 요청 수신, 실행 처리, 상태 보고

### ScenarioLibraryComponent
- ScenarioLibrary
- 책임: 시나리오 정의 및 저장소 관리, 시나리오 인스턴스 생성

### ScenarioExecutionComponent
- ScenarioExecutionManager
- IScenarioExecutor 인터페이스
- 시나리오 구현체들 (PickPlaceScenario, DoorOperationScenario 등)
- 책임: 시나리오 실행 로직 정의 및 실행 흐름 관리

### CommandBrokerComponent
- CommandBroker
- 책임: command_executor 노드와 통신, 명령 전달 및 결과 수신

### ExecutionStateComponent
- ExecutionStateTracker
- 책임: 시나리오 실행 상태 추적 및 보고

### ExternalInterfaceComponent
- IScenarioReceiver 구현체
- 책임: 상위 시스템(ACS, API 서버)으로부터 시나리오 요청 수신

---

# 2. 컴포넌트 상세 설명

## TaskManagerNodeComponent
- **구성 요소**: `TaskManagerNode`
- **역할**:
  - 메인 ROS2 노드로서 전체 시스템 조율
  - 상위 시스템으로부터 시나리오 요청 수신
  - 시나리오 실행 흐름 관리
  - 상태 보고 및 피드백 제공
- **입력**:
  - handle_scenario_request(scenario_type: str, params: Dict)
  - cancel_current_scenario()
  - get_execution_status()
- **출력**:
  - 시나리오 실행 결과
  - 실시간 상태 업데이트
- **협력**:
  - ScenarioLibraryComponent
  - ScenarioExecutionComponent
  - CommandBrokerComponent
  - ExecutionStateComponent

## ScenarioLibraryComponent
- **구성 요소**: `ScenarioLibrary`
- **역할**:
  - 시스템에서 지원하는 모든 시나리오 유형 관리
  - 시나리오 인스턴스 생성 및 관리
  - 시나리오 종속성 및 사용 가능성 확인
- **입력**:
  - get_scenario(scenario_type: str)
  - register_scenario(scenario_type: str, executor_class)
  - check_scenario_availability(scenario_type: str)
- **출력**:
  - 시나리오 실행기 인스턴스
  - 시나리오 가용성 정보
- **협력**:
  - TaskManagerNodeComponent
  - ScenarioExecutionComponent

## ScenarioExecutionComponent
- **구성 요소**: `ScenarioExecutionManager`, `IScenarioExecutor` 및 구현체들
- **역할**:
  - 시나리오 실행 로직 정의
  - 시나리오 유형별 특화된 실행 전략 구현
  - 명령 시퀀스 생성 및 순차적 실행
  - 명령 완료 결과에 따른 다음 단계 결정
- **입력**:
  - execute_scenario(scenario, params)
  - handle_command_result(result)
  - cancel_execution()
  - pause_execution()
  - resume_execution()
- **출력**:
  - 명령 시퀀스
  - 시나리오 실행 진행 상태
- **협력**:
  - TaskManagerNodeComponent
  - CommandBrokerComponent
  - ExecutionStateComponent

## CommandBrokerComponent
- **구성 요소**: `CommandBroker`
- **역할**:
  - command_executor 노드와의 통신 담당
  - 명령 요청 전송 및 결과 수신
  - 명령 상태 모니터링
  - 비동기 명령 실행 지원
- **입력**:
  - send_command(command_type, params)
  - cancel_command(command_id)
  - check_command_status(command_id)
  - wait_for_result(command_id, timeout)
- **출력**:
  - 명령 ID
  - 명령 실행 결과 및 상태
- **협력**:
  - ScenarioExecutionComponent
  - command_executor_node (외부)

## ExecutionStateComponent
- **구성 요소**: `ExecutionStateTracker`
- **역할**:
  - 시나리오 실행 상태 추적
  - 진행 상황 모니터링
  - 상태 정보 발행 및 로깅
- **입력**:
  - update_state(state)
  - publish_state()
  - log_execution_progress()
- **출력**:
  - 현재 실행 상태 정보
  - 진행률 데이터
- **협력**:
  - ScenarioExecutionComponent
  - TaskManagerNodeComponent

## ExternalInterfaceComponent
- **구성 요소**: `IScenarioReceiver` 구현체
- **역할**:
  - 상위 시스템(ACS, API 서버)으로부터 시나리오 요청 수신
  - 요청 데이터 유효성 검증
  - 요청을 TaskManagerNode가 처리할 수 있는 형태로 변환
- **입력**:
  - receive_scenario_request(request_data)
  - validate_request(request_data)
- **출력**:
  - 시나리오 요청 객체
  - 유효성 검증 결과
- **협력**:
  - TaskManagerNodeComponent
  - 외부 시스템 (ACS, FastAPI 서버)

---

# 3. 컴포넌트 간 관계 요약

- `ExternalInterfaceComponent` → `TaskManagerNodeComponent` (시나리오 요청 전달)
- `TaskManagerNodeComponent` → `ScenarioLibraryComponent` (시나리오 인스턴스 요청)
- `TaskManagerNodeComponent` → `ScenarioExecutionComponent` (시나리오 실행 요청)
- `ScenarioExecutionComponent` → `CommandBrokerComponent` (명령 전송 요청)
- `CommandBrokerComponent` → `command_executor_node` (실제 명령 송신, ROS2 서비스 호출)
- `ScenarioExecutionComponent` → `ExecutionStateComponent` (상태 업데이트)
- `TaskManagerNodeComponent` → `ExecutionStateComponent` (상태 조회/발행)

---

# 4. 요약

| 컴포넌트 | 책임 | 협력 |
|:--|:--|:--|
| TaskManagerNodeComponent | 전체 시스템 조율, 시나리오 관리 | 모든 컴포넌트, 외부 시스템 |
| ScenarioLibraryComponent | 시나리오 정의 및 인스턴스 관리 | TaskManager, ScenarioExecution |
| ScenarioExecutionComponent | 시나리오 실행 및 명령 시퀀스 관리 | TaskManager, CommandBroker, ExecutionState |
| CommandBrokerComponent | command_executor와 통신, 명령 전달 | ScenarioExecution, command_executor |
| ExecutionStateComponent | 상태 추적 및 보고 | TaskManager, ScenarioExecution |
| ExternalInterfaceComponent | 외부 시스템 요청 수신 | TaskManager, 외부 시스템 |

---

# 5. 주요 시나리오 흐름 예시

1. 상위 시스템(ACS/FastAPI)에서 'pick_place' 시나리오 실행 요청
2. ExternalInterfaceComponent가 요청을 수신하고 검증
3. TaskManagerNodeComponent가 요청을 처리하기 위해 ScenarioLibraryComponent에 시나리오 요청
4. ScenarioLibraryComponent가 PickPlaceScenario 인스턴스 생성
5. TaskManagerNodeComponent가 ScenarioExecutionComponent에 실행 요청
6. ScenarioExecutionComponent가 시나리오 실행 시작:
   - 명령 시퀀스(move → pick → move → place) 생성
   - CommandBrokerComponent를 통해 순차적으로 명령 전송
   - 각 명령 완료 후 다음 명령 실행
7. ExecutionStateComponent가 실행 상태 추적 및 발행
8. 모든 명령 완료 후 최종 결과 반환

---