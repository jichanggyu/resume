## command_executor 패키지 고급 설계 문서 (최종 수정본)

---

### 1. 책임 (Responsibility)

`command_executor` 패키지는 task_manager로부터 받은 개별 명령을 처리하고,
하드웨어 수준의 명령으로 변환하여 h/w_node에 전달하는 미들웨어 역할을 담당한다.

- task_manager로부터 명령(pick, place, move 등) 수신
- 명령 유형에 따라 적절한 핸들러로 처리 위임
- 명령을 하위 수준의 하드웨어 명령 시퀀스로 변환
- h/w_node와 통신하여 실제 하드웨어 제어 명령 전송
- 명령 실행 상태 모니터링 및 결과 반환
- 명령 취소 및 오류 상황 대응

---

### 2. 패키지 의존 관계 (Dependencies)

| 연결 대상 패키지 | 연결 방식 | 설명 |
|------------------|-----------|------|
| `task_manager` | ROS2 Service Server | 명령 요청 수신 대상 |
| `h_w_node` | ROS2 Service Client | 하드웨어 수준 명령 전송 |
| `status_node` | ROS2 Topic Publish | 명령 실행 상태 정보 발행 |
| `safety_interlock_node` | ROS2 Topic Subscribe | 안전 관련 이벤트 수신 및 대응 |
| `vision_node` | ROS2 Service Client (선택적) | 필요시 비전 정보 요청 |

---

### 3. 외부 인터페이스 정의 (Interfaces)

#### 제공하는 ROS2 서비스

| 서비스명 | 타입 | 설명 |
|----------|------|------|
| `/command_executor/execute_command` | `ExecuteCommand.srv` | 특정 명령 실행 요청 처리 |
| `/command_executor/check_status` | `CheckCommandStatus.srv` | 명령 실행 상태 확인 |
| `/command_executor/cancel_command` | `CancelCommand.srv` | 실행 중인 명령 취소 |

#### 발행하는 ROS2 토픽

| 토픽명 | 타입 | 설명 |
|--------|------|------|
| `/command_executor/command_status` | `CommandStatus.msg` | 현재 명령 실행 상태 정보 |
| `/command_executor/command_result` | `CommandResult.msg` | 명령 실행 결과 정보 |

#### 사용하는 ROS2 서비스 (클라이언트)

| 서비스명 | 타입 | 설명 |
|----------|------|------|
| `/hw_node/move_arm` | `MoveArm.srv` | 로봇 팔 이동 명령 |
| `/hw_node/control_gripper` | `ControlGripper.srv` | 그리퍼 제어 명령 |
| `/hw_node/check_status` | `CheckHWStatus.srv` | 하드웨어 상태 확인 |

---

### 4. 내부 구성 요소 (컴포넌트 구조)

| 컴포넌트명 | 타입 | 책임 |
|------------|------|------|
| `CommandExecutorNode` | ROS2 Node | 명령 수신 및 처리, ROS2 서비스 제공 |
| `CommandDispatcher` | Class | 명령 유형에 따른 핸들러 선택 및 실행 |
| `HardwareNodeConnector` | Class | h/w_node와 통신, 하드웨어 명령 전송 |
| `CommandExecutionContext` | Data Class | 명령 실행 컨텍스트 및 상태 관리 |
| `CommandResponsePublisher` | Class | 명령 상태 및 결과 발행 |
| `ICommandHandler` | Interface | 모든 명령 핸들러의 기본 인터페이스 |
| `IHardwareConnector` | Interface | 하드웨어 노드 연결 인터페이스 |

### 명령 핸들러 구현체
- `PickCommandHandler` (implements `ICommandHandler`)
- `PlaceCommandHandler` (implements `ICommandHandler`)
- `MoveCommandHandler` (implements `ICommandHandler`)
- `GripperOpenCommandHandler` (implements `ICommandHandler`)
- `GripperCloseCommandHandler` (implements `ICommandHandler`)
- `DoorOpenCommandHandler` (implements `ICommandHandler`)
- `DoorCloseCommandHandler` (implements `ICommandHandler`)
- `StopCommandHandler` (implements `ICommandHandler`)

---

### 5. 동작 흐름 예시

1. task_manager로부터 `/command_executor/execute_command` 서비스 요청 수신 (명령: "pick")
2. `CommandExecutorNode`가 요청을 처리하여 명령 ID 생성 및 `CommandExecutionContext` 초기화
3. `CommandDispatcher`가 "pick" 명령을 인식하고 `PickCommandHandler` 선택
4. `PickCommandHandler`는 실행 시작:
   - 파라미터 검증
   - 다단계 하드웨어 명령 시퀀스 생성 (접근 → 그리퍼 제어 → 파지 등)
   - `HardwareNodeConnector`를 통해 `/hw_node/move_arm`, `/hw_node/control_gripper` 등 서비스 호출
   - 각 하드웨어 명령 완료 확인 후 다음 단계 진행
5. `CommandResponsePublisher`를 통해 명령 상태 및 진행 상황 토픽 발행
6. 모든 하드웨어 명령 완료 후 최종 결과 생성
7. 성공/실패 여부에 따라 결과 반환 및 task_manager에 통보

---

### 6. 명령 실행 예시 - Pick

1. Pick 명령 수신 (object_id: "box_1")
2. PickCommandHandler 실행 시작
3. 하드웨어 명령 시퀀스 생성:
   - `/hw_node/move_arm` 호출 (접근 위치로 이동)
   - vision_node에서 물체 정확한 위치 획득
   - `/hw_node/move_arm` 호출 (물체 직전 위치로 이동)
   - `/hw_node/control_gripper` 호출 (그리퍼 열기)
   - `/hw_node/move_arm` 호출 (물체 파지 위치로 이동)
   - `/hw_node/control_gripper` 호출 (그리퍼 닫기)
   - 파지 성공 확인
   - `/hw_node/move_arm` 호출 (안전 위치로 이동)
4. 각 단계 완료 시 상태 업데이트 및 발행
5. 최종 결과 생성 및 반환

---

### 7. 예외/에러 상황 대응

| 상황 | 대응 방법 |
|------|----------|
| 잘못된 명령 매개변수 | 유효성 검증 후 오류 반환 |
| 지원하지 않는 명령 유형 | 오류 코드와 메시지 반환 |
| H/W node 응답 없음 | 타임아웃 후 실패 처리, 재연결 시도 |
| 명령 실행 중 취소 요청 | 즉시 H/W 명령 취소 후 상태 업데이트 |
| H/W node 오류 응답 | 오류 코드 변환 후 task_manager에 전달 |
| 안전 이벤트 수신 | 모든 명령 즉시 취소 및 안전 상태 전환 |

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

- task_manager로부터 명령을 수신하고 h/w_node로 전달하는 중개자 역할 명확화
- 개별 명령 유형별 전용 핸들러로 책임 분리
- 하드웨어 명령 시퀀스 생성 및 순차적 실행 로직 구현
- 명령 실행 상태 추적 및 결과 피드백 시스템
- 안전하고 유연한 명령 실행 구조
- 다양한 예외 상황에 대한 처리 메커니즘 확립
