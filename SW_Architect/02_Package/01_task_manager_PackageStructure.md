## task_manager 패키지 고급 설계 문서 (최종 수정본)

---

### 1. 책임 (Responsibility)

`task_manager` 패키지는 상위 시스템으로부터 받은 요청에 따라 적절한 시나리오를 선택하고 
시나리오를 구성하는 개별 명령들을 command_executor에게 순차적으로 전달하는 역할을 담당한다.

- 상위 시스템으로부터 시나리오 유형 수신 (배송, 물건 정리, 물체 이동 등)
- 시나리오 유형에 따른 작업 시퀀스 생성 및 관리
- 시나리오를 구성하는 개별 명령을 command_executor에게 순차적 전달
- 시나리오 실행 상태 모니터링 및 관리
- 시나리오 간 전환 및 우선순위 관리
- 작업 실패 시 복구 또는 대체 시나리오 실행
- 상위 시스템에 상태 보고 및 UI/사용자 피드백 제공

---

### 2. 패키지 의존 관계 (Dependencies)

| 연결 대상 패키지 | 연결 방식 | 설명 |
|------------------|-----------|------|
| `command_executor` | ROS2 Service Client | 개별 명령 실행 요청 |
| `status_node` | ROS2 Topic Subscribe | 로봇 상태 모니터링 |
| `acs_interface` | ROS2 Service Server | 상위 시스템 요청 수신 |
| `safety_interlock_node` | ROS2 Topic Subscribe | 안전 관련 이벤트 수신 |

---

### 3. 외부 인터페이스 정의 (Interfaces)

#### 제공하는 ROS2 서비스

| 서비스명 | 타입 | 설명 |
|----------|------|------|
| `/task_manager/start_scenario` | `StartScenario.srv` | 시나리오 실행 요청 처리 |
| `/task_manager/stop_scenario` | `StopScenario.srv` | 진행 중인 시나리오 중지 |
| `/task_manager/pause_scenario` | `PauseScenario.srv` | 시나리오 일시 중지 |
| `/task_manager/resume_scenario` | `ResumeScenario.srv` | 일시 중지된 시나리오 재개 |
| `/task_manager/get_status` | `GetStatus.srv` | 현재 시나리오 상태 조회 |

#### 발행하는 ROS2 토픽

| 토픽명 | 타입 | 설명 |
|--------|------|------|
| `/task_manager/scenario_status` | `ScenarioStatus.msg` | 시나리오 실행 상태 정보 |
| `/task_manager/scenario_feedback` | `ScenarioFeedback.msg` | 시나리오 실행 피드백 |

#### 사용하는 ROS2 서비스 (클라이언트)

| 서비스명 | 타입 | 설명 |
|----------|------|------|
| `/command_executor/execute_command` | `ExecuteCommand.srv` | 명령 실행 요청 |
| `/command_executor/check_status` | `CheckCommandStatus.srv` | 명령 상태 확인 |
| `/command_executor/cancel_command` | `CancelCommand.srv` | 명령 취소 요청 |

---

### 4. 내부 구성 요소 (컴포넌트 구조)

| 컴포넌트명 | 타입 | 책임 |
|------------|------|------|
| `TaskManagerNode` | ROS2 Node | 시나리오 요청 수신 및 전체 실행 흐름 관리 |
| `ScenarioLibrary` | Class | 시나리오 템플릿 관리 및 인스턴스화 |
| `ScenarioExecutionManager` | Class | 시나리오 실행 및 상태 추적 |
| `CommandSequencer` | Class | 명령 시퀀스 생성 및 관리 |
| `ScenarioStateTracker` | Class | 시나리오 상태 추적 및 업데이트 |
| `CommandExecutorConnector` | Class | command_executor와 통신 담당 |
| `StatusListener` | Class | 로봇 상태 변화 감지 및 대응 |

### 시나리오 구현체
- `DeliveryScenario` (extends `BaseScenario`)
- `CleanupScenario` (extends `BaseScenario`)
- `ItemTransferScenario` (extends `BaseScenario`)
- `DoorManagementScenario` (extends `BaseScenario`)
- `ChargingScenario` (extends `BaseScenario`)

---

### 5. 동작 흐름 예시

1. `/task_manager/start_scenario` 서비스 요청 수신 (시나리오: "delivery")
2. `TaskManagerNode`가 요청을 받아 `ScenarioLibrary`에서 "delivery" 시나리오 템플릿 로드
3. `ScenarioExecutionManager`가 시나리오 인스턴스 생성 및 초기화
4. `CommandSequencer`가 시나리오에 맞는 명령 시퀀스 생성
5. 각 명령을 순차적으로:
   - `CommandExecutorConnector`를 통해 `/command_executor/execute_command` 호출
   - 명령 완료 대기 및 결과 확인
   - 다음 명령으로 진행 또는 오류 처리
6. `ScenarioStateTracker`가 상태 변화 추적 및 토픽 발행
7. 시나리오 완료 후 결과 생성 및 응답

---

### 6. 시나리오 실행 예시 - 배송

1. "delivery" 시나리오 요청 수신 (target_location: "room_101")
2. 배송 시나리오 명령 시퀀스 생성:
   - 현재 위치에서 물건 보관소로 이동 (Move 명령)
   - 물건 인식 및 집기 (Pick 명령)
   - 목적지로 이동 (Move 명령)
   - 문 열기 (DoorOpen 명령)
   - 방 안으로 진입 (Move 명령)
   - 물건 내려놓기 (Place 명령)
   - 방에서 나오기 (Move 명령)
   - 문 닫기 (DoorClose 명령)
   - 대기 위치로 이동 (Move 명령)
3. 각 명령마다:
   - command_executor에 명령 요청
   - 상태 및 결과 모니터링
   - 다음 단계 진행 결정
4. 시나리오 완료 상태 발행 및 상위 시스템에 통보

---

### 7. 예외/에러 상황 대응

| 상황 | 대응 방법 |
|------|----------|
| 지원하지 않는 시나리오 요청 | 오류 메시지 반환 및 거부 |
| 명령 실행 실패 | 재시도 또는 대체 경로 탐색 |
| 시나리오 중지 요청 | 진행 중인 명령 취소 후 시나리오 종료 |
| 안전 이벤트 수신 | 모든 명령 취소 및 안전 상태로 전환 |
| 로봇 상태 변화 감지 | 시나리오 실행 조정 (속도 변경, 대기 등) |
| 연결 끊김 | 재연결 시도 및 복구 단계 실행 |

---

### 8. task_manager와 command_executor, h/w node 역할 구분

| task_manager | command_executor | h/w node |
|:--|:--|:--|
| 시나리오 유형을 선택하고 관리 | 개별 명령을 H/W 명령으로 변환 | 하드웨어 직접 제어 |
| 전체 작업 흐름 제어 | 다단계 H/W 명령 시퀀스 생성 | 저수준 하드웨어 명령 실행 |
| 시나리오 간 전환 결정 | 명령 실행 상태 모니터링 | 센서 데이터 수집 및 처리 |
| 상위 시스템 요청 처리 | H/W 노드 오류 처리 및 재시도 | 안전 기능 구현 |
| UI/사용자 피드백 제공 | 완료/실패 결과 생성 및 반환 | 하드웨어 상태 모니터링 |

---

### 9. 최종 요약

- 상위 시스템과 command_executor 간의 중개자 역할 명확화
- 시나리오 라이브러리 기반 유연한 작업 흐름 관리
- 명령 시퀀스 생성 및 순차적 실행 관리
- 다양한 예외 상황 대응 및 복구 메커니즘
- 실시간 상태 모니터링 및 피드백 시스템
- 시나리오 간 우선순위 관리 및 전환 기능

```yaml
scenario_name: pick_and_place
steps:
  - action: move
    params: { target: "bin1" }
  - action: pick
    params: { object_id: "A123" }
  - action: move
    params: { target: "tray" }
  - action: place
    params: { location: "tray" }
