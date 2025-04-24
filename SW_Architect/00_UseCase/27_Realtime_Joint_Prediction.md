# UC27 – 실시간 로봇 조인트 예측 요청 (ROS2 기반 통신 구조)

## 개요

| 항목 | 내용 |
|------|------|
| **유즈케이스 ID** | UC27 |
| **유즈케이스 명** | 실시간 로봇 조인트 예측 요청 |
| **관련 기능 요구사항** | F5-1. 로봇 행동 예측 기능 |
| **주 액터** | Vision_Node, Status_Node, AI_Node (AI PC), Status_Node or Command_Executor_Node |
| **목적** | 로봇의 실시간 이미지와 조인트 상태를 ROS2 네트워크를 통해 AI 노드로 전송하고, 다음 조인트 상태를 예측 받아 후속 처리에 활용한다. |

---

## 사전 조건

- `Vision_Node`는 ROS2를 통해 실시간 이미지(`/image_raw`)를 퍼블리시하고 있어야 한다.
- `Status_Node`는 `/joint_states` 토픽을 통해 로봇의 현재 조인트 상태를 퍼블리시 중이어야 한다.
- `AI_Node`는 ROS2 subscriber로 등록되어 있어야 하며, 추론 결과를 `/predicted_joint` 토픽에 퍼블리시할 수 있어야 한다.
- ROS2 통신 환경은 AI PC와 Backend PC 간에 QoS 및 네트워크가 안정적으로 구성되어야 한다.

---

## 후 조건

- AI PC에서 추론된 다음 조인트 값이 ROS2 토픽(`/predicted_joint`)으로 퍼블리시된다.
- 후속 노드(command_executor 등)가 해당 값을 구독하거나, FastAPI를 통해 UI로 전달된다.
- 추론 결과는 상태 확인 또는 실제 동작 실행 판단에 활용될 수 있다.

---

## 기본 흐름

1. `Vision_Node`가 실시간으로 카메라 이미지(`/image_raw`)를 퍼블리시한다.
2. `Status_Node`가 현재 로봇의 `/joint_states`를 퍼블리시한다.
3. `AI_Node`는 `/image_raw`와 `/joint_states`를 동시에 구독하여 입력값으로 사용한다.
4. `AI_Node`는 다음 시점의 예측된 조인트 값(예: [1.01, -0.53, 0.75, ...])을 `/predicted_joint` 토픽에 퍼블리시한다.
5. `Status_Node` 또는 `Command_Executor_Node`가 `/predicted_joint`를 구독하여 결과를 상태 처리 또는 실행 판단에 활용한다.
6. FastAPI는 필요 시 `/predicted_joint` 값을 WebSocket 등을 통해 Web_UI에 표시한다.

---

## 예외 흐름

- **E1: AI 노드가 연결되지 않음**  
  `/image_raw` 또는 `/joint_states` 구독 실패 → 로그 기록, fallback 모드 진입

- **E2: 추론 지연 또는 예측 실패**  
  AI 연산이 일정 시간 초과 또는 결과 미반환 → 이전 조인트값 유지 또는 skip

- **E3: 통신 지연**  
  ROS2 네트워크 QoS 설정 오류 또는 bandwidth 부족 → 이미지 프레임 누락, 예측 정확도 저하 가능

---

## 통신 흐름

| 구간 | 방식 | 내용 |
|------|------|------|
| Vision_Node → AI_Node | ROS2 Topic (`/image_raw`) | 실시간 이미지 프레임 (CompressedImage 또는 Image) |
| Status_Node → AI_Node | ROS2 Topic (`/joint_states`) | 현재 조인트 상태 벡터 |
| AI_Node → 전체 시스템 | ROS2 Topic (`/predicted_joint`) | 예측된 다음 조인트 상태 |
| FastAPI → Web_UI (옵션) | WebSocket | `/ws/predicted_joint` 등으로 UI 표시용 전달 |

---

## 비고

- ROS2 기반 구조는 이미지 + 조인트 벡터를 병렬로 처리하는 데 최적화되어 있음.
- 추론 주기 유지(30Hz)를 위해 AI_Node 내부에 buffer, queue, frame drop 제한 등 필요
- 추론 결과가 실제 로봇 제어에 반영될 경우, 별도의 안전 판단 로직 및 제한 조건 필수

