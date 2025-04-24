# UC20 – 웹 UI를 통해 로봇 제어 명령 전송 (Send Robot Command via Web UI)

## 개요

| 항목 | 내용 |
|------|------|
| **유즈케이스 ID** | UC20 |
| **유즈케이스 명** | 웹 UI를 통해 로봇 제어 명령 전송 |
| **관련 기능 요구사항** | SR6. 웹 기반 제어 UI 제공 |
| **주 액터** | 운영자 (Web 브라우저 사용자) |
| **목적** | 웹 UI를 통해 로봇에게 Pick, Place, Door 제어, 이동 등의 명령을 전달한다. |

---

## 사전 조건

- 로봇 시스템과 Web UI 간 통신 연결이 정상 상태여야 한다.
- ROS2 노드 및 FastAPI 서버가 정상 구동되어 있어야 한다.

## 후 조건

- 명령 전송 성공 시 로봇이 지정된 동작을 수행한다.
- 명령 실행 결과(성공/실패)가 사용자에게 즉시 표시된다.

---

## 기본 흐름

1. 사용자가 Web UI에서 원하는 제어 명령 버튼을 클릭한다.
2. Web UI는 FastAPI 서버로 RestFul POST 요청을 전송한다.
3. FastAPI 서버는 해당 명령 요청을 ROS2 서비스 호출로 변환하여 Command Executor Node에 전달한다.
4. Command Executor Node는 명령을 해석하여 하드웨어로 실행 명령을 발행한다.
5. 실행 결과를 FastAPI에 회신한다.
6. FastAPI는 결과를 Web UI로 변환하여 사용자에게 표시한다.

---

## 예외 흐름

- **E1: API 통신 오류**  
  FastAPI 서버가 응답하지 않을 경우  
  → “명령 전송 실패” 팝업 또는 경고 표시

- **E2: ROS2 응답 지연/실패**  
  → 일정 시간 동안 결과가 수신되지 않으면 "명령 미응답" 상태 표시

---

## 통신 흐름

| 송신자 | 수신자 | 방식 | 메시지 |
|--------|--------|------|--------|
| Web UI | FastAPI | WebSocket or REST | /api/robot/{command} |
| FastAPI | Schedular Node | ROS2 Service Call | /Schedular_node/execute_command |
| Schedular Node | Command Executor Node | 내부 ROS2 통신 (Service/Request) | 명령 실행 요청 |
| Command Executor Node | H/W Node | 내부 ROS2 통신 (Service/Request) | 하드웨어 제어 명령 송신 |

---

## 비고

- 각 명령의 UI 컴포넌트는 명확한 피드백(로딩, 결과, 실패)을 제공해야 함
- 네트워크 지연에 대한 대처 메시지(로딩 표시, 재전송 버튼 등)도 포함 추천