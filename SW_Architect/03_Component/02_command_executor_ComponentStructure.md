# command_executor 패키지 – Component Structure (최종 수정본)

---

## 1. 컴포넌트 목록

### CommandExecutorNodeComponent
- CommandExecutorNode
- 책임: task_manager로부터 명령을 수신하고, 적절한 명령 처리 흐름 제어

### CommandDispatchComponent
- CommandDispatcher
- 책임: 명령 유형에 따라 적절한 핸들러를 선택하고 실행 위임

### CommandHandlerComponent
- ICommandHandler 인터페이스
- 다양한 명령 핸들러 구현체들:
  - PickCommandHandler
  - PlaceCommandHandler
  - MoveCommandHandler
  - GripperOpenCommandHandler
  - GripperCloseCommandHandler
  - DoorOpenCommandHandler
  - DoorCloseCommandHandler
  - StopCommandHandler
- 책임: 개별 명령 실행 로직 구현 및 H/W 노드 명령으로 변환

### HardwareConnectorComponent
- HardwareNodeConnector
- IHardwareConnector 인터페이스
- 책임: H/W 노드와의 통신 담당, 명령 전송 및 결과 수신

### CommandContextComponent
- CommandExecutionContext
- 책임: 명령 실행 컨텍스트 및 상태 관리

### ResponsePublisherComponent
- CommandResponsePublisher
- 책임: 명령 실행 결과 및 상태 발행

---

## 2. 컴포넌트 상세 설명

### CommandExecutorNodeComponent
- **구성 요소**: CommandExecutorNode
- **역할**:
  - ROS2 노드로서 task_manager로부터 명령 수신
  - 명령 실행 흐름 제어 및 조율
  - 명령 실행 결과 반환
  - 실행 상태 모니터링 및 보고
- **입력**:
  - `+ handle_command(command_type: str, params: Dict) -> CommandID`
  - `+ check_status(command_id: CommandID) -> CommandStatus`
  - `+ cancel_command(command_id: CommandID) -> bool`
- **출력**:
  - 명령 실행 ID
  - 명령 상태 및 결과
- **협력**:
  - CommandDispatchComponent
  - HardwareConnectorComponent
  - ResponsePublisherComponent

### CommandDispatchComponent
- **구성 요소**: CommandDispatcher
- **역할**:
  - 명령 유형에 따라 적절한 핸들러를 선택
  - 핸들러 등록 및 관리
  - 활성 명령 추적
- **입력**:
  - `+ dispatch(command_type: str, params: Dict) -> CommandID`
  - `+ register_handler(command_type: str, handler: ICommandHandler) -> bool`
  - `+ get_active_command(command_id: CommandID) -> ICommandHandler`
- **출력**:
  - 명령 실행 ID
  - 선택된 핸들러 인스턴스
- **협력**:
  - CommandHandlerComponent
  - CommandContextComponent

### CommandHandlerComponent
- **구성 요소**: 
  - ICommandHandler 인터페이스
  - 다양한 명령 핸들러 구현체들
- **역할**:
  - 각 명령 유형별 실행 로직 구현
  - 명령 파라미터 검증
  - 하드웨어 명령 시퀀스 생성
  - 명령 실행 및 취소 처리
- **입력**:
  - `+ execute(context: CommandExecutionContext) -> CommandResult`
  - `+ validate_params(params: CommandParams) -> bool`
  - `+ cancel() -> bool`
- **출력**:
  - 명령 실행 결과
  - 검증 결과
- **협력**:
  - HardwareConnectorComponent
  - CommandContextComponent

### HardwareConnectorComponent
- **구성 요소**: HardwareNodeConnector, IHardwareConnector
- **역할**:
  - H/W 노드와의 통신 담당
  - ROS2 서비스 클라이언트 관리
  - 명령 전송 및 결과 수신
  - 명령 상태 모니터링
- **입력**:
  - `+ send_command(hw_node: str, command: str, params: Dict) -> HWCommandID`
  - `+ check_command_status(command_id: HWCommandID) -> HWCommandStatus`
  - `+ cancel_command(command_id: HWCommandID) -> bool`
- **출력**:
  - 하드웨어 명령 ID
  - 하드웨어 명령 상태 및 결과
- **협력**:
  - CommandHandlerComponent
  - H/W 노드 (외부)

### CommandContextComponent
- **구성 요소**: CommandExecutionContext
- **역할**:
  - 명령 실행 컨텍스트 관리
  - 명령 실행 상태 추적
  - 매개변수 및 결과 저장
  - 관련 하드웨어 명령 ID 추적
- **속성**:
  - command_id
  - command_type
  - params
  - hw_command_ids
  - status
  - result
- **협력**:
  - CommandHandlerComponent
  - CommandExecutorNodeComponent

### ResponsePublisherComponent
- **구성 요소**: CommandResponsePublisher
- **역할**:
  - 명령 실행 결과 및 상태 발행
  - 비동기 상태 업데이트 지원
  - ROS2 토픽 발행 관리
- **입력**:
  - `+ publish_status(command_id: CommandID, status: CommandStatus) -> None`
  - `+ publish_result(command_id: CommandID, result: CommandResult) -> None`
- **출력**:
  - 상태 메시지
  - 결과 메시지
- **협력**:
  - CommandExecutorNodeComponent

---

## 3. 컴포넌트 간 관계 요약

- CommandExecutorNodeComponent → CommandDispatchComponent (명령 분배 요청)
- CommandExecutorNodeComponent → CommandContextComponent (컨텍스트 생성 및 관리)
- CommandExecutorNodeComponent → ResponsePublisherComponent (결과 발행 요청)
- CommandDispatchComponent → CommandHandlerComponent (명령 실행 요청)
- CommandHandlerComponent → HardwareConnectorComponent (하드웨어 명령 전송)
- CommandHandlerComponent → CommandContextComponent (컨텍스트 사용 및 업데이트)
- HardwareConnectorComponent → 외부 H/W 노드 (ROS2 서비스 호출)

---

## 4. 요약

| 컴포넌트 | 책임 | 협력 |
|:--|:--|:--|
| CommandExecutorNodeComponent | task_manager 명령 수신 및 처리 | Dispatcher, HardwareConnector, ResponsePublisher |
| CommandDispatchComponent | 명령 유형별 핸들러 선택 및 실행 | CommandHandler, Context |
| CommandHandlerComponent | 명령별 실행 로직 구현, H/W 명령 변환 | HardwareConnector, Context |
| HardwareConnectorComponent | H/W 노드와 통신, 명령 전송 | 외부 H/W 노드, CommandHandler |
| CommandContextComponent | 명령 컨텍스트 및 상태 관리 | 모든 핸들러, CommandExecutor |
| ResponsePublisherComponent | 명령 결과 및 상태 발행 | CommandExecutor, task_manager |

---

## 5. 주요 명령 처리 흐름 예시

1. task_manager로부터 "pick" 명령 수신
2. CommandExecutorNodeComponent가 명령 ID 생성 및 CommandContextComponent 초기화
3. CommandDispatchComponent가 PickCommandHandler 선택
4. PickCommandHandler가 실행 시작:
   - 파라미터 검증
   - 다단계 하드웨어 명령 시퀀스 생성 (팔 이동 → 그리퍼 제어 등)
   - HardwareConnectorComponent를 통해 순차적으로 H/W 명령 전송
   - 각 H/W 명령 완료 후 다음 단계 진행
5. 명령 진행 상태를 CommandContextComponent에 지속 업데이트
6. ResponsePublisherComponent가 주기적으로 상태 발행
7. 모든 H/W 명령 완료 후 최종 결과 설정 및 반환

---

## 6. task_manager ↔ command_executor ↔ h/w node 관계

- **task_manager**:
  - 시나리오 유형 선택 및 관리
  - 명령 시퀀스 생성 및 조율
  - 고수준 작업 관리

- **command_executor**:
  - 개별 명령 실행 처리
  - 명령을 H/W 수준으로 변환
  - 하드웨어 통신 관리

- **h/w node**:
  - 실제 하드웨어 직접 제어
  - 저수준 하드웨어 명령 실행
  - 센서 데이터 수집 및 상태 모니터링

---
