# 🧩 vision_node 패키지 – Class Diagram (최종 수정본)

---

# ✅ 1. 클래스 목록 및 구조

### 📘 Interface (추상 클래스)
- `IImageProcessor` (이미지 처리 인터페이스)
- `IFeatureExtractor` (특징점 추출 인터페이스)
- `IGraspPointGenerator` (그래스프 포인트 생성 인터페이스)

### 📘 Core Classes
- `VisionNode` (rclpy.Node, 비전 처리 노드)
- `ImageProcessingPipeline` (이미지 처리 파이프라인)
- `FeatureExtractionManager` (특징점 추출 관리)
- `GraspPointManager` (그래스프 포인트 관리)
- `VisionDataPublisher` (비전 데이터 발행)
- `HalconInterface` (Halcon 소프트웨어 인터페이스)

### 📘 구현체
- `EdgeDetector` (implements IImageProcessor)
- `ColorSegmentation` (implements IImageProcessor)
- `HalconFeatureExtractor` (implements IFeatureExtractor, Halcon 활용)
- `HalconGraspGenerator` (implements IGraspPointGenerator, Halcon 활용)

---

# ✅ 2. 인터페이스 상세 설명

## 📘 IImageProcessor
- **책임**: 이미지 전처리 및 기본 처리
- **행동**:
  - `+ process(image: np.ndarray) -> np.ndarray`
  - `+ set_parameters(params: dict) -> None`
- **협력**: ImageProcessingPipeline

## 📘 IFeatureExtractor
- **책임**: 이미지에서 특징점 추출
- **행동**:
  - `+ extract_features(image: np.ndarray) -> List[Feature]`
  - `+ match_features(features1: List[Feature], features2: List[Feature]) -> List[Match]`
- **협력**: FeatureExtractionManager

## 📘 IGraspPointGenerator
- **책임**: 그래스프 포인트 생성
- **행동**:
  - `+ generate_grasp_points(image: np.ndarray, features: List[Feature]) -> List[GraspPoint]`
  - `+ rank_grasp_points(points: List[GraspPoint]) -> List[GraspPoint]`
- **협력**: GraspPointManager

---

# ✅ 3. Core Class 상세 설명

## 📘 VisionNode
- **책임**: 비전 시스템 전체 관리
- **속성**:
  - `- processing_pipeline: ImageProcessingPipeline`
  - `- feature_manager: FeatureExtractionManager`
  - `- grasp_manager: GraspPointManager`
  - `- halcon_interface: HalconInterface`
- **행동**:
  - `+ process_image(image: np.ndarray) -> dict`
  - `+ extract_features(image: np.ndarray) -> List[Feature]`
  - `+ generate_grasp_points(image: np.ndarray) -> List[GraspPoint]`

## 📘 ImageProcessingPipeline
- **책임**: 이미지 처리 파이프라인 관리
- **속성**:
  - `- processors: List[IImageProcessor]`
- **행동**:
  - `+ add_processor(processor: IImageProcessor) -> None`
  - `+ process_image(image: np.ndarray) -> np.ndarray`
  - `+ get_intermediate_results() -> List[np.ndarray]`

## 📘 FeatureExtractionManager
- **책임**: 특징점 추출 관리
- **속성**:
  - `- feature_extractor: IFeatureExtractor`
  - `- halcon_interface: HalconInterface`
- **행동**:
  - `+ extract_features(image: np.ndarray) -> List[Feature]`
  - `+ configure_extractor(config: dict) -> bool`

## 📘 GraspPointManager
- **책임**: 그래스프 포인트 생성 및 관리
- **속성**:
  - `- grasp_generator: IGraspPointGenerator`
  - `- halcon_interface: HalconInterface`
- **행동**:
  - `+ generate_grasp_points(image: np.ndarray, features: List[Feature]) -> List[GraspPoint]`
  - `+ get_best_grasp_point() -> GraspPoint`

## 📘 VisionDataPublisher
- **책임**: 비전 처리 결과 발행
- **행동**:
  - `+ publish_image(image: np.ndarray) -> None`
  - `+ publish_features(features: List[Feature]) -> None`
  - `+ publish_grasp_points(points: List[GraspPoint]) -> None`

## 📘 HalconInterface
- **책임**: Halcon 소프트웨어와의 통합 인터페이스
- **속성**:
  - `- halcon_engine: object`
  - `- procedures: Dict[str, object]`
- **행동**:
  - `+ initialize() -> bool`
  - `+ run_procedure(name: str, params: dict) -> dict`
  - `+ extract_features(image: np.ndarray) -> List[Feature]`
  - `+ generate_grasp(image: np.ndarray, features: List[Feature]) -> List[GraspPoint]`

---

# ✅ 4. 구현체 상세 설명

## 📘 EdgeDetector
- **책임**: 이미지에서 에지 추출
- **행동**:
  - `+ process(image: np.ndarray) -> np.ndarray`
  - `+ set_parameters(threshold: float, sigma: float) -> None`

## 📘 ColorSegmentation
- **책임**: 색상 기반 이미지 분할
- **행동**:
  - `+ process(image: np.ndarray) -> np.ndarray`
  - `+ set_parameters(color_range: dict) -> None`

## 📘 HalconFeatureExtractor
- **책임**: Halcon을 활용한 특징점 추출
- **행동**:
  - `+ extract_features(image: np.ndarray) -> List[Feature]`
  - `+ configure(params: dict) -> None`
- **비고**: Halcon의 특징점 추출 알고리즘 활용

## 📘 HalconGraspGenerator
- **책임**: Halcon을 활용한 그래스프 포인트 생성
- **행동**:
  - `+ generate_grasp_points(image: np.ndarray, features: List[Feature]) -> List[GraspPoint]`
  - `+ configure(params: dict) -> None`
- **비고**: Halcon의 그래스핑 알고리즘 활용

---

# ✅ 5. 클래스 간 관계 요약 (UML 관계)

- VisionNode → (Aggregation) → ImageProcessingPipeline, FeatureExtractionManager, GraspPointManager, HalconInterface
- ImageProcessingPipeline → (Association) → IImageProcessor 구현체들
- FeatureExtractionManager → (Association) → IFeatureExtractor
- GraspPointManager → (Association) → IGraspPointGenerator
- HalconFeatureExtractor → (Association) → HalconInterface
- HalconGraspGenerator → (Association) → HalconInterface

---

# ✅ 6. ROS2 및 시스템 특성 반영

- VisionNode는 다음 서비스/토픽 제공:
  - `/vision/process_image`
  - `/vision/extract_features`
  - `/vision/generate_grasp_points`
- 이미지 처리 결과 실시간 발행
- 특징점 및 그래스프 포인트 시각화 데이터 발행
- Halcon 기반 고급 비전 처리 지원

---

# ✅ 7. 예외/에러 상황 대응

| 상황 | 대응 방법 |
|:--|:--|
| 이미지 수신 실패 | 에러 로그 및 상태 알림 |
| 이미지 처리 실패 | 재시도 또는 에러 보고 |
| 특징점 추출 실패 | Halcon 대체 알고리즘 시도 |
| 그래스프 생성 실패 | 이전 성공한 결과 사용 |
| Halcon 연결 실패 | 내부 백업 알고리즘으로 전환 |

---

# 📢 최종 요약

- 하드웨어 제어와 비전 처리의 책임 분리
- Halcon 소프트웨어 통합으로 고급 비전 기능 활용
- 특징점 추출 및 그래스프 포인트 생성 기능 강화
- 확장 가능한 이미지 처리 파이프라인
- UseCase 요구사항 완벽 반영

---
