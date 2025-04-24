## 🧩 acs_interface 패키지 고급 설계 문서 (최종 수정본)

---

### 1. 📌 책임 (Responsibility)

`acs_interface` 패키지는 로봇 시스템과 상위 제어 시스템(ACS, AMR 등) 간의 통신을 담당하는 외부 인터페이스 계층이다.  
명령 수신 및 상태 보고를 모두 담당하며, 주요 책임은 다음과 같다:

- ACS로부터 작업 명령 수신 (시작, 중지, 시나리오 요청 등)
- 작업 명령을 task_manager로 전달
- 로봇의 상태 정보를 주기적으로 또는 이벤트 기반으로 ACS에 전송
- 연결 유지 및 통신 상태 확인
- WebSocket 기반의 양방향 통신 제공

---

### 2. 🔗 패키지 의존 관계 (Dependencies)

| 연결 대상 패키지 | 연결 방식 | 설명 |
|------------------|-----------|------|
| `task_manager` | ROS2 Service 호출 | 수신된 ACS 명령을 작업 흐름으로 전달 |
| `status_reporter` | ROS2 Topic 구독 | 현재 로봇 상태를 받아와 ACS에 전송 |
| `safety_interlock_node` | ROS2 Topic 구독 | 안전 관련 이벤트 수신 및 전파 |
| `ui_interface` | (간접 연결) | ACS 상태를 UI에서도 공유 가능 |

---

### 3. 📬 외부 인터페이스 정의 (Interfaces)

#### ✅ 수신: ACS → 로봇

| 방식 | 포맷 | 설명 |
|------|------|------|
| WebSocket (JSON) | `{ "command": "start", "scenario": "A" }` | 작업 시작 요청 |
| WebSocket (JSON) | `{ "command": "stop" }` | 로봇 동작 중단 |
| WebSocket (JSON) | `{ "command": "status_request" }` | 현재 상태 요청 |
| WebSocket (JSON) | `{ "command": "modify_scenario", "modifications": {...} }` | 시나리오 수정 |

#### ✅ 송신: 로봇 → ACS

| 방식 | 포맷 | 설명 |
|------|------|------|
| WebSocket (JSON) | `{ "status": "idle", "pose": {...}, "battery": 87.5 }` | 현재 상태 정기 보고 |
| WebSocket (JSON) | `{ "event": "error", "detail": "collision" }` | 이벤트 알림 보고 |
| WebSocket (JSON) | `{ "ack": "received", "command": "start" }` | 명령 수신 응답 |
| WebSocket (JSON) | `{ "safety_alert": true, "reason": "emergency_stop" }` | 안전 관련 알림 |

---

### 4. ⚙️ 내부 구성 요소 (컴포넌트 구조)

| 컴포넌트명 | 타입 | 역할 |
|------------|------|------|
| `ACSInterfaceNode` | ROS2 Node | WebSocket 기반으로 ACS와 통신 관리 |
| `WebSocketClient` | Class | WebSocket 연결 및 메시지 송수신 처리 |
| `ACSCommandReceiver` | Class | ACS로부터의 명령 수신, 파싱 및 분배 |
| `ACSStatusReporter` | Class | 로봇 상태 수집 및 ACS로 전송 |

---

### 5. 🔄 동작 흐름 예시

#### 작업 명령 수신 흐름
1. ACS → `{ "command": "start", "scenario": "B" }` 메시지 WebSocket으로 전송
2. `WebSocketClient`가 메시지 수신하여 `ACSInterfaceNode`에 전달
3. `ACSInterfaceNode`가 `ACSCommandReceiver`의 `parse_and_dispatch` 메소드 호출
4. `ACSCommandReceiver`가 명령 파싱 후 `task_manager`의 해당 서비스 호출
5. 결과를 `{ "ack": "received", "command": "start" }` 형태로 ACS에 응답

#### 상태 보고 흐름
1. `ACSStatusReporter`가 주기적으로 `gather_status()` 메소드 호출
2. `status_reporter` 패키지에서 발행하는 토픽 데이터 수집
3. 수집된 상태 정보를 JSON 형태로 변환
4. `WebSocketClient`를 통해 상태 정보를 ACS로 전송

---

### 6. 📊 클래스 간 관계 및 상호작용

- ACSInterfaceNode는 WebSocketClient, ACSCommandReceiver, ACSStatusReporter를 포함(Aggregation)
- ACSCommandReceiver는 task_manager와 연결(Association)하여 명령 전달
- ACSStatusReporter는 status_reporter와 연결하여 상태 정보 수집

---

### 7. ⚠️ 예외/에러 상황 대응

| 상황 | 대응 방법 |
|------|----------|
| WebSocket 연결 실패 | 자동 재연결 시도 및 에러 로깅 |
| 메시지 파싱 실패 | 무시하고 경고 로그 출력 |
| 상태 전송 실패 | 다음 주기까지 대기 후 재전송 시도 |
| 통신 두절 | 타임아웃 감지 및 연결 복구 시도 |
| 잘못된 명령 수신 | 오류 메시지 응답 및 로깅 |

---

### 8. 📢 최종 요약

- WebSocket 기반의 양방향 통신으로 ACS와 효율적 인터페이스 제공
- 명령 수신과 상태 보고 기능을 전문 클래스로 분리하여 모듈화
- 주기적인 상태 보고 및 이벤트 기반 알림 체계 구현
- 연결 관리 및 복구 기능 내장
- 다양한 예외 상황 대응 메커니즘 구현