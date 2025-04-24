## 🧩 ui_interface_node 패키지 고급 설계 문서 (최종 수정본)

---

### 1. 📌 책임 (Responsibility)

`ui_interface_node` 패키지는 사용자 웹 인터페이스(React)와 로봇 시스템 간의 연결을 담당하는 FastAPI 기반 패키지이다.  
사용자의 입력을 ROS2 시스템에 전달하고, 실시간 상태를 UI에 반영하는 중간 브리지 역할을 수행한다.

- 사용자 요청 수신 (정지, 이동, Pick/Place 등)
- REST API 경로 제공 (GET/POST 요청 처리)
- 상태 변화 시 WebSocket을 통해 실시간 전송
- ROS2 서비스 호출 및 토픽 구독
- 다중 사용자 연결 지원 및 상태 동기화
- 인터페이스 기반 설계로 확장성 제공

---

### 2. 🔗 패키지 의존 관계 (Dependencies)

| 연결 대상 패키지 | 연결 방식 | 설명 |
|------------------|-----------|------|
| `task_manager` | ROS2 서비스 호출 | `/scheduler/select_scenario`, `/scheduler/start` 등 |
| `status_reporter` | ROS2 토픽 구독 | 로봇 상태 정보 수신 (위치, 배터리, 통신 등) |
| `command_executor` | ROS2 서비스 호출 | Pick/Place/Move 등 명령 전달 |
| `safety_interlock_node` | ROS2 토픽 구독 | 안전 관련 상태 및 알림 수신 |
| `robot_collision_detection` | ROS2 토픽 구독 | 충돌 위험 정보 수신 |

---

### 3. 📬 외부 인터페이스 정의 (Interfaces)

#### ✅ 제공하는 REST API

| 엔드포인트 | 메서드 | 설명 |
|------------|--------|------|
| `/api/robot/stop` | POST | 로봇 동작 정지 |
| `/api/robot/move_pose` | POST | 좌표로 이동 |
| `/api/command/pick` | POST | Pick 명령 |
| `/api/command/place` | POST | Place 명령 |
| `/api/scenario/select` | POST | 시나리오 선택 및 적용 |
| `/api/status/pose` | GET | 현재 위치 정보 조회 |
| `/api/status/battery` | GET | 배터리 상태 조회 |
| `/api/status/task` | GET | 현재 실행 중인 작업 조회 |
| `/api/status/safety` | GET | 안전 관련 상태 조회 |

#### ✅ WebSocket 채널

| 경로 | 설명 |
|------|------|
| `/ws/status` | 전체 상태 정보 실시간 push (pose, battery, task 등) |
| `/ws/alerts` | 알람 및 에러 상태 실시간 전달 |
| `/ws/visualization` | 시각화 데이터 실시간 전달 |

---

### 4. ⚙️ 내부 구성 요소 (컴포넌트 구조)

#### 인터페이스 (추상 클래스)
- `ICommandService` (명령 실행 서비스 인터페이스)
- `IStatusService` (상태 조회 서비스 인터페이스)
- `IScenarioService` (시나리오 선택 서비스 인터페이스)
- `IWebSocketManager` (WebSocket 연결 관리 인터페이스)

#### 핵심 클래스
| 컴포넌트명 | 타입 | 역할 |
|------------|------|------|
| `APIServer` | FastAPI 인스턴스 | 전체 서버 엔트리 및 API 엔드포인트 등록 |
| `CommandRouter` | Class | 명령 관련 HTTP 엔드포인트 처리 |
| `StatusRouter` | Class | 상태 조회 관련 HTTP 엔드포인트 처리 |
| `ScenarioRouter` | Class | 시나리오 선택 엔드포인트 처리 |
| `CommandService` | Class (implements ICommandService) | 명령을 ROS2 서비스로 전송 |
| `StatusService` | Class (implements IStatusService) | ROS2 Topic으로부터 상태 수신 |
| `ScenarioService` | Class (implements IScenarioService) | 시나리오 선택 ROS2 서비스 호출 |
| `WebSocketManager` | Class (implements IWebSocketManager) | WebSocket 연결 관리 및 메시지 브로드캐스트 |
| `StatusBroadcaster` | Class | 로봇 상태 업데이트를 WebSocket으로 전송 |

---

### 5. 🔄 동작 흐름 예시

#### REST API 요청 처리 흐름
1. 사용자 → `/api/robot/pick` POST 요청
2. `CommandRouter`가 요청을 수신하여 검증
3. `CommandService`의 `send_pick_command` 메소드 호출
4. `CommandService`가 ROS2 서비스 클라이언트를 통해 `command_executor`의 서비스 호출
5. 실행 결과를 HTTP 응답으로 클라이언트에 반환

#### 상태 업데이트 및 전파 흐름
1. `StatusService`가 ROS2 토픽을 통해 상태 변경 수신
2. 수신된 상태 정보를 내부 캐시에 업데이트
3. `StatusBroadcaster`가 상태 변경 감지
4. `WebSocketManager`를 통해 연결된 모든 클라이언트에 상태 업데이트 메시지 브로드캐스트
5. 클라이언트 UI에 실시간으로 상태 반영

---

### 6. 📊 클래스 간 관계 및 상호작용

- APIServer는 CommandRouter, StatusRouter, ScenarioRouter를 포함(Aggregation)
- CommandRouter는 ICommandService와 연결(Association)하여 명령 실행
- StatusRouter는 IStatusService와 연결하여 상태 조회
- ScenarioRouter는 IScenarioService와 연결하여 시나리오 선택
- CommandService, StatusService, ScenarioService는 각각 대응하는 인터페이스를 구현(Implements)
- StatusBroadcaster는 WebSocketManager를 포함하여 상태 브로드캐스트
- WebSocketManager는 IWebSocketManager를 구현하여 WebSocket 연결 관리

---

### 7. ⚠️ 예외/에러 상황 대응

| 상황 | 대응 방법 |
|------|----------|
| ROS2 서비스 호출 실패 | HTTP 500 에러 반환 및 로깅 |
| WebSocket 연결 실패 | 연결 해제 후 재시도 안내 |
| 요청 파라미터 오류 | HTTP 400 에러 반환 및 상세 오류 메시지 제공 |
| 상태 수신 실패 | 캐시된 이전 데이터 반환 또는 오류 상태 표시 |
| 다중 클라이언트 연결 충돌 | 세션 관리로 충돌 방지 |
| 서버 과부하 | 요청 제한 및 대기열 관리 |

---

### 8. 📢 최종 요약

- REST API와 WebSocket 이중 통신 체계로 다양한 상호작용 지원
- 인터페이스 기반 설계로 서비스 로직 분리 및 확장성 확보
- ROS2 서비스 호출 및 토픽 구독을 통합한 브리지 역할 수행
- 상태 캐싱 및 실시간 브로드캐스트로 효율적인 상태 동기화
- 다중 사용자 지원 및 충돌 방지 메커니즘 구현
- 다양한 예외 상황 대응으로 안정적인 서비스 제공