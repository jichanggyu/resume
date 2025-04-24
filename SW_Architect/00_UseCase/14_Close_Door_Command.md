# UC14 – Door Close 명령 (Close Door Command)

## 개요

| 항목 | 내용 |
|------|------|
| **유즈케이스 ID** | UC14 |
| **유즈케이스 명** | Door Close 명령 |
| **관련 기능 요구사항** | F5-3. Door Close |
| **주 액터** | 운영자 (Web UI 사용자) |
| **목적** | 로봇이 장비 또는 구조물의 도어를 닫도록 제어한다. |

---

## 사전 조건

- 도어가 열린 상태여야 한다 (센서 또는 내부 상태 기반).
- 도어의 닫힘 위치 및 궤적이 사전에 정의되어 있어야 한다.
- 로봇 팔이 도어 닫힘 위치까지 안전하게 이동 가능해야 한다.

## 후 조건

- 도어가 완전히 닫히고, 상태가 "닫힘"으로 UI에 표시된다.
- 이후 동작이 안전하게 수행될 수 있도록 대기 상태로 전환된다.

---

## 기본 흐름

1. 운영자가 Web UI에서 Door Close 명령을 실행하거나, Schedular Node가 해당 명령을 내린다.
2. Command Executor Node는 Door Close 동작을 시작한다.
3. H/W Node에 도어 닫기 동작 수행을 요청한다.
4. H/W Node는 로봇팔을 통해 도어를 밀거나 끌어 닫는 시퀀스를 수행한다.
5. 도어가 완전히 닫혔는지 확인 센서 또는 위치 기반 판단으로 결과를 확인한다.
6. Status Node가 결과를 발행하고, FastAPI를 통해 Web UI에 전달된다.

---

## 예외 흐름

- **E1: 도어 이미 닫힘 상태**  
  → “도어가 이미 닫혀 있습니다” 메시지 표시

- **E2: 도어 닫기 궤적 오류**  
  → 경로 계산 실패 또는 도달 불가능  
  → “닫기 실패: 위치 오류” 메시지 출력

- **E3: 물체 끼임 감지**  
  → 힘 센서 또는 토크 이상 감지  
  → 즉시 동작 중지 및 경고 “이물질 감지” 표시

---

## 통신 흐름

| 송신자 | 수신자 | 방식 | 메시지 |
|--------|--------|------|--------|
| Schedular Node or UI | Command Executor Node | 내부 명령 or ROS2 Topic | Door Close 명령 |
| Command Executor Node | H/W Node | ROS2 Action or Topic | 도어 닫기 동작 실행 |
| H/W Node | Status Node | ROS2 Topic | "도어 닫힘 성공/실패" 상태 발행 |
| Status Node | FastAPI | ROS2 Topic | `/robot/status` |
| FastAPI | Web UI | WebSocket or 응답 | 결과 표시 |

---

## 비고

- 닫힘 완료 여부는 힘 센서, 위치 센서, 시간 제한 등으로 검증 가능
- 안전을 위해 닫기 속도 조절 및 충돌 감지 필수
- 필요 시 자동 도어 보정 시나리오와 연계 가능
