# vision_node 패키지 – Component Structure (최종 수정본)

---

## 1. 컴포넌트 목록

### VisionNodeComponent
- VisionNode
- 책임: ROS2 인터페이스 제공 및 전체 비전 시스템 조정

### ImageProcessingComponent
- ImageProcessingPipeline
- IImageProcessor 구현체들 (EdgeDetector, ColorSegmentation 등)
- 책임: 이미지 전처리 및 처리 파이프라인 관리

### FeatureExtractionComponent
- FeatureExtractionManager
- IFeatureExtractor 구현체들 (HalconFeatureExtractor 등)
- 책임: 이미지에서 특징점 추출 수행

### GraspPointGenerationComponent
- GraspPointManager
- IGraspPointGenerator 구현체들 (HalconGraspGenerator 등)
- 책임: 특징점 기반 그래스프 포인트 생성

### HalconIntegrationComponent
- HalconInterface
- 책임: Halcon 소프트웨어와의 통합 인터페이스 제공

### DataPublishingComponent
- VisionDataPublisher
- 책임: 비전 처리 결과 발행 및 시각화

---

## 2. 컴포넌트 상세 설명

### VisionNodeComponent
- **구성 요소**: VisionNode
- **역할**:
  - ROS2 노드로서 서비스 및 토픽 인터페이스 제공
  - 전체 비전 처리 흐름 조정
  - 다른 컴포넌트들의 실행 순서 관리
  - 에러 처리 및 실패 복구 메커니즘 제공
- **입력**:
  - 서비스 요청 (/vision/process_image, /vision/extract_features, /vision/generate_grasp_points)
  - 카메라로부터의 이미지 데이터
- **출력**:
  - 처리 결과 및 상태
  - 진단 메시지 및 오류 정보
- **협력**:
  - 모든 비전 처리 컴포넌트
  - 다른 ROS2 노드와의 상호작용

### ImageProcessingComponent
- **구성 요소**: ImageProcessingPipeline, IImageProcessor 구현체들
- **역할**:
  - 이미지 전처리 및 필터링
  - 다양한 처리 단계를 파이프라인으로 구성
  - 중간 처리 결과 관리 및 제공
- **입력**:
  - `+ process_image(image: np.ndarray) -> np.ndarray`
  - `+ add_processor(processor: IImageProcessor) -> None`
  - `+ get_intermediate_results() -> List[np.ndarray]`
- **출력**:
  - 처리된 이미지
  - 중간 결과의 시퀀스
- **협력**:
  - VisionNodeComponent
  - FeatureExtractionComponent

### FeatureExtractionComponent
- **구성 요소**: FeatureExtractionManager, IFeatureExtractor 구현체들
- **역할**:
  - 이미지로부터 특징점 추출
  - 다양한 특징 추출 알고리즘 관리
  - 특징점 매칭 및 분석
- **입력**:
  - `+ extract_features(image: np.ndarray) -> List[Feature]`
  - `+ configure_extractor(config: dict) -> bool`
  - `+ match_features(features1: List[Feature], features2: List[Feature]) -> List[Match]`
- **출력**:
  - 특징점 목록
  - 특징점 매칭 결과
- **협력**:
  - VisionNodeComponent
  - ImageProcessingComponent
  - HalconIntegrationComponent

### GraspPointGenerationComponent
- **구성 요소**: GraspPointManager, IGraspPointGenerator 구현체들
- **역할**:
  - 특징점을 기반으로 그래스프 포인트 생성
  - 최적의 파지 위치 계산
  - 그래스프 포인트 품질 평가 및 순위 지정
- **입력**:
  - `+ generate_grasp_points(image: np.ndarray, features: List[Feature]) -> List[GraspPoint]`
  - `+ get_best_grasp_point() -> GraspPoint`
  - `+ rank_grasp_points(points: List[GraspPoint]) -> List[GraspPoint]`
- **출력**:
  - 그래스프 포인트 목록
  - 품질 순위가 지정된 그래스프 포인트
  - 최적 그래스프 포인트
- **협력**:
  - VisionNodeComponent
  - FeatureExtractionComponent
  - HalconIntegrationComponent

### HalconIntegrationComponent
- **구성 요소**: HalconInterface
- **역할**:
  - Halcon 소프트웨어와의 통합 인터페이스 제공
  - 고급 비전 알고리즘 실행
  - Halcon 프로시저 관리 및 실행
- **입력**:
  - `+ initialize() -> bool`
  - `+ run_procedure(name: str, params: dict) -> dict`
  - `+ extract_features(image: np.ndarray) -> List[Feature]`
  - `+ generate_grasp(image: np.ndarray, features: List[Feature]) -> List[GraspPoint]`
- **출력**:
  - Halcon 처리 결과
  - Halcon 기반 특징점 및 그래스프 데이터
- **협력**:
  - FeatureExtractionComponent
  - GraspPointGenerationComponent

### DataPublishingComponent
- **구성 요소**: VisionDataPublisher
- **역할**:
  - 비전 처리 결과 발행
  - 결과 시각화 데이터 생성
  - 진단 정보 발행
- **입력**:
  - `+ publish_image(image: np.ndarray) -> None`
  - `+ publish_features(features: List[Feature]) -> None`
  - `+ publish_grasp_points(points: List[GraspPoint]) -> None`
- **출력**:
  - ROS2 토픽 메시지
  - 시각화 마커 데이터
- **협력**:
  - VisionNodeComponent
  - 다른 비전 컴포넌트들 (결과 발행용)
  - RViz 등 시각화 도구와의 인터페이스

---

## 3. 컴포넌트 간 관계 요약

- **VisionNodeComponent**는 전체 비전 처리 흐름을 조정하며 다음과 같은 관계를 가집니다:
  - → **ImageProcessingComponent**: 이미지 처리 요청 및 설정 관리
  - → **FeatureExtractionComponent**: 특징점 추출 요청 및 설정 관리
  - → **GraspPointGenerationComponent**: 그래스프 포인트 생성 요청 및 설정 관리
  - → **DataPublishingComponent**: 결과 발행 요청

- **ImageProcessingComponent**는 이미지 처리 파이프라인을 구성하고 다음과 같은 관계를 가집니다:
  - → **FeatureExtractionComponent**: 처리된 이미지 전달
  - ← **VisionNodeComponent**: 처리 요청 및 설정 수신

- **FeatureExtractionComponent**는 특징점 추출을 담당하고 다음과 같은 관계를 가집니다:
  - → **HalconIntegrationComponent**: Halcon 기반 특징점 추출 기능 활용
  - → **GraspPointGenerationComponent**: 추출된 특징점 전달
  - ← **ImageProcessingComponent**: 처리된 이미지 수신
  - ← **VisionNodeComponent**: 추출 요청 및 설정 수신

- **GraspPointGenerationComponent**는 그래스프 포인트 생성을 담당하고 다음과 같은 관계를 가집니다:
  - → **HalconIntegrationComponent**: Halcon 기반 그래스프 생성 기능 활용
  - ← **FeatureExtractionComponent**: 특징점 수신
  - ← **VisionNodeComponent**: 생성 요청 및 설정 수신

- **HalconIntegrationComponent**는 Halcon 소프트웨어와의 인터페이스를 제공하고 다음과 같은 관계를 가집니다:
  - ← **FeatureExtractionComponent**: Halcon 특징점 추출 요청 수신
  - ← **GraspPointGenerationComponent**: Halcon 그래스프 생성 요청 수신

- **DataPublishingComponent**는 처리 결과를 발행하고 다음과 같은 관계를 가집니다:
  - ← **VisionNodeComponent**: 발행 요청 수신
  - ← **모든 결과 생성 컴포넌트**: 발행할 데이터 수신

---

## 4. 에러 처리 및 복구 전략

| 상황 | 담당 컴포넌트 | 처리 방법 |
|:--|:--|:--|
| 이미지 수신 실패 | VisionNodeComponent | 에러 로그 기록, 재시도 메커니즘 실행, 상태 알림 |
| 이미지 처리 실패 | ImageProcessingComponent | 대체 처리 파이프라인 시도, VisionNodeComponent에 오류 보고 |
| 특징점 추출 실패 | FeatureExtractionComponent | 대체 알고리즘 시도, 이전 프레임 데이터 활용 |
| 그래스프 생성 실패 | GraspPointGenerationComponent | 이전 성공한 결과 사용, 대체 전략 시도 |
| Halcon 연결 실패 | HalconIntegrationComponent | 내부 백업 알고리즘으로 전환, 재연결 시도 |
| 데이터 발행 실패 | DataPublishingComponent | 로컬 캐싱, 재시도 메커니즘 실행 |

---

## 5. 요약

| 컴포넌트 | 책임 | 협력 |
|:--|:--|:--|
| VisionNodeComponent | ROS2 인터페이스 및 전체 조정 | 모든 비전 컴포넌트 |
| ImageProcessingComponent | 이미지 전처리 및 파이프라인 관리 | Vision, Feature |
| FeatureExtractionComponent | 특징점 추출 및 매칭 | Vision, Image, Halcon |
| GraspPointGenerationComponent | 그래스프 포인트 생성 및 평가 | Vision, Feature, Halcon |
| HalconIntegrationComponent | Halcon 소프트웨어 인터페이스 | Feature, GraspPoint |
| DataPublishingComponent | 결과 발행 및 시각화 | Vision, 결과 생성 컴포넌트들 |

---

## 6. 최종 정리

- 모듈식 비전 처리 시스템으로 높은 확장성 제공
- 이미지 처리, 특징점 추출, 그래스프 생성의 논리적 분리를 통한 관리 용이성
- Halcon 소프트웨어 통합으로 고급 비전 기능 활용 가능
- 파이프라인 기반의 유연한 이미지 처리 프로세스
- 인터페이스를 통한 다양한 알고리즘 플러그인 지원
- ROS2 기반의 표준화된 발행-구독 구조
- 확장 및 변경에 대응하는 유연한 설계
- 견고한 에러 처리 및 복구 메커니즘

---
