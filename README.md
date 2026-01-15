# 3D_objectdetection
![Team_Logo](./background.png)

# AICV_3D-Object-Detection
3D Object Detection with NuscenesDataset(Mini), Modelname: centerPoint

# 1. 주제 및 선정배경

---

RGB 영상 기반 2D object detection은 객체를 이미지 평면 상에서만 인식하기 때문에 실제 거리 정보를 직접적으로 제공하지 못하며, 원근, 조명 변화, 그림자, 가림 현상 등에 취약하다. 이러한 요인들은 자율주행 환경에서 객체의 실제 위치와 크기를 정확하게 판단하는 데 큰 오차를 유발할 수 있다. 

반면 LiDAR는 레이저를 이용해 거리 정보를 직접 측정하므로 조명 조건에 영향을 받지 않으며, 물체의 3차원 공간적 구조를 정확하게 파악할 수 있다. 이로 인해 객체의 위치, 크기, 방향을 보다 정밀하게 추정할 수 있으며, 충돌 회피와 같은 안전 관련 판단에 필수적인 정보를 제공한다. 따라서 실제 자율주행 시스템에서는 LiDAR 기반 3D object detection이 핵심적인 환경 인지 수단으로 활용된다.

<img width="585" height="172" alt="image" src="https://github.com/user-attachments/assets/e5dd0b15-5a5f-482a-8436-245fdfcf7fdd" />

# 데이터셋: nuScenes mini

---
<img width="1589" height="787" alt="image" src="https://github.com/user-attachments/assets/62de67da-2b38-44ff-970e-e39d56f4a853" />


**데이터셋 선정 배경**

- **KITTI 대신 nuScenes 데이터셋을 선정한 이유**
    - KITTI의 한계: KITTI는 전면(Front-view)에 제한된 시야각만을 제공하여 초기의 연구에는 유용했으나 복잡한 교차로 주행이나 차선 변경(Cut-in)과 같은 실제 자율주행 시나리오를 대변하지 못함
    - nuScenes: nuScenes 데이터셋은 업계 최초로 360도 전방위 라이다(LiDAR) 및 카메라 데이터를 제공하여 전방위(Surround-view) 3D 객체 탐지 표준을 정립하여 과거에는 운전자 시점(Camera View)에서 분석했지만, nuScenes 이후로 하늘에서 내려다보는 시점인 BEV으로 데이터를 변환하여 처리하도록 해주었음
    - 360도 사각지대 없는 탐지와 복잡한 도심 트래픽을 직접 다뤄보는 것이 좀 더 유의미하다고 판단하여 nuScenes를 채택하게 되었음
- **Original (Full) vs Mini**
    - Full  데이터셋은 학습에 수일이 소요되는 반면, Mini는 대표적인 10개의 scenes만 추린 버전이지만, 데이터 포맷은 완전히 동일함. 따라서 제한된 리소스 내에서 초기 파이프 라인 구축과 모델 아키텍처의 유효성 검증을 신속히 구축/검증하기 위해 Mini 데이터셋을 채택하여 적용함
    - 이후 Original (Full)로 스케일업하여 대규모 본 학습을 수행하고, 재현 가능한 정량 평가 및 결과 리포팅으로 연구 결과를 고도화할 계획
 

# 문제 및 해결

### 문제상황

### MMDetection3D 환경 구축 중 Config 상속 문제 및 데이터 버전 불일치

### 1. 문제 상황 (Problem)

NuScenes **Mini 데이터셋(v1.0-mini)**을 사용하여 **Pointpillars**, **CenterPoint** 모델을 학습시키는 과정에서 두 가지 치명적인 오류가 발생하여 학습이 중단됨.

1. **Wrapper 클래스 인자 오류 (`TypeError`)**: 데이터 불균형 해소를 위한 `CBGSDataset` 래퍼(Wrapper)가 내부 데이터셋용 인자(`data_root`, `ann_file` 등)를 처리하지 못해 에러 발생.
2. **데이터 버전 불일치 (`AssertionError`)**: Config 파일 내 평가자(Evaluator)가 하드코딩된 `v1.0-trainval` 경로를 요구했으나, 실제 환경에는 `v1.0-mini`만 존재하여 검증(Validation) 단계에서 실패.
3. **버전 호환성:** MMDet3D의 Config 파일 설정과 내부 소스 코드 간의 `version` 인자 전달 과정에서 충돌 발생. 소스 코드 패치 등을 시도했으나 라이브러리 의존성 문제로 불안정함.

### 2. 원인 분석 (Root Cause Analysis)

- **구조적 원인**: MMDetection3D의 Config 시스템은 `dataset`을 `CBGSDataset`으로 감싸는 구조임. Config Loader가 전역 변수(`data_root` 등)를 최상위 클래스인 `CBGSDataset`에 주입하려 했으나, 해당 클래스에는 이를 받는 `__init__` 인자가 정의되어 있지 않음.
- **라이브러리 제약**: `NuScenesMetric` 클래스 내부 로직이 특정 버전 폴더명(`v1.0-trainval`)을 필수적으로 요구하도록 설계되어 있어, 단순 Config 수정(`version='v1.0-mini'`)만으로는 해결되지 않는 호환성 문제가 있음.

### MMDetection3D 기반 CenterPoint 구현 과정에서의 데이터 스케일 및 환경 구성 이슈

<img width="1260" height="234" alt="image" src="https://github.com/user-attachments/assets/4f392e01-7d9b-4b7c-86de-c3c57f78f257" />


<img width="1880" height="860" alt="image" src="https://github.com/user-attachments/assets/516cec9a-e814-4329-8bad-e2949cfb5ded" />


1. **데이터 규모와 모델 간의 괴리 (Data Scale Mismatch)**
대규모 데이터 학습에 최적화된 CenterPoint 모델을 소량의 Mini 데이터셋에 무리하게 적용하다 보니, **데이터 절대량 부족으로 인한 과소적합(Underfitting)**이 발생했습니다.
    1. PointPillars는 "대규모 데이터 학습 자체"보다는 "추론 속도(Inference Speed)와 효율성"에 최적화된 아키텍처
2. **하이퍼파라미터 최적화 부족 (Optimization Issues)**
전체 데이터셋 기준인 '20 Epoch' 설정을 그대로 사용하여 학습량이 턱없이 부족했으며, **데이터 규모에 맞춰 Epoch를 대폭 늘리는 등 세밀한 튜닝**을 하지 못한 아쉬움이 있습니다.
3. **클래스 불균형에 따른 성능 저하 (Class Imbalance)**
Mini 데이터셋 특성상 특정 클래스 데이터가 거의 없어 모델이 학습할 기회를 얻지 못했고, 이것이 **해당 객체의 검출 실패와 전체적인 성능 저하**로 이어졌습니다.

### 해결방안

- **방법 1: 클래스 수 줄이기 → 전체 데이터셋을 Mini용으로 최적화**
    
    Mini 데이터셋에 **확실히 많이 존재하는 클래스만** 남기고 나머지는 Config에서 지우세요. 보통 `car`, `pedestrian` 2개만 남겨도 결과가 훨씬 깔끔해집니다.
    
    1. **`class_names` 수정:**Python
        
        `# 기존 10개 다 쓰지 말고, 있는 것만 남김
        class_names = ['car', 'pedestrian']`
        
    2. **`tasks` 수정:**Python
        
        `tasks=[
            dict(num_class=1, class_names=['car']),
            dict(num_class=1, class_names=['pedestrian']),
        ]`
        
        (*나머지 Head 설정도 이에 맞춰 줄여야 합니다.*)
        
- **방법 2: Pre-trained Weight 사용 (`load_from`) → 전체 데이터셋을 Mini용으로 최적화**
    
    Mini로 학습을 시키더라도, 이미 다른 거대한 데이터(Waymo, NuScenes Full)로 똑똑해진 모델을 가져와서 **Fine-tuning**을 해야 합니다.
    
    - MMDetection3D Model Zoo에서 `centerpoint_voxel0075_second_secfpn_8xb4-cyclic-20e_nus-3d`의 `.pth` 파일을 다운로드 받으세요.
    - Config 파일에 다음을 추가합니다:Python
        
        `load_from = 'path/to/centerpoint_checkpoint.pth'`
        
        이렇게 하면 이미 자동차를 알아보는 눈을 가진 상태에서 Mini 데이터에 적응만 하므로 결과가 잘 나옵니다.
        
- **방법 3 : 데이터 스케일 및 환경 구성 이슈 해결 → 성능 개선으로 가진 않음. (학습하기 위한 최소 조건으로 만들어줌)**
    
    ### 1. 해결 과정 (Solution)
    
    **Step 1: 동적 Config 수정을 통한 래퍼(Wrapper) 구조 정상화**
    
    - Config 파일을 직접 수정하는 대신, Python 스크립트 내에서 `Config` 객체를 로드하여 **런타임에 구조를 변경**하는 방식을 채택.
    - `CBGSDataset` (껍데기)에 잘못 주입된 인자들을 제거(`pop`)하고, 이를 내부의 실제 데이터셋(`dataset.dataset`)으로 올바르게 재할당하는 **"Config 수술(Surgery)" 스크립트**를 작성하여 자동화함.
    
    **Step 2: 심볼릭 링크(Symbolic Link)를 활용한 경로 우회**
    
    - 라이브러리의 소스 코드를 직접 수정하는 것은 유지보수 측면에서 좋지 않으므로(Hard-coding 지양), **파일 시스템 레벨**에서 해결책을 모색.
    - `os.symlink`를 사용하여 `v1.0-mini` 폴더를 가리키는 `v1.0-trainval` **심볼릭 링크(바로가기)**를 생성.
    - 이를 통해 평가자(Evaluator)는 `trainval` 데이터가 있다고 인식하게 하여, 코드 수정 없이 **논리적 정합성**을 유지하며 검증 단계를 통과시킴.
    
    ### 2. 핵심 코드 (Key Code Snippet)
    
    ```python
    Python
    
    # 1. Config 객체 동적 수정 (Wrapper 문제 해결)
    if cfg.train_dataloader.dataset.type == 'CBGSDataset':
        # 래퍼가 처리 못하는 키 제거
        banned_keys = ['data_root', 'ann_file', 'data_prefix', 'pipeline']
        for key in banned_keys:
            cfg.train_dataloader.dataset.pop(key, None)
        
        # 내부 데이터셋에 올바른 경로 주입
        cfg.train_dataloader.dataset.dataset.data_root = data_root
    
    # 2. 심볼릭 링크 생성 (버전 불일치 해결)
    src = os.path.join(data_dir, "v1.0-mini")
    dst = os.path.join(data_dir, "v1.0-trainval") # Evaluator가 찾는 이름
    if not os.path.exists(dst):
        os.symlink(src, dst) # Mini를 Trainval인 것처럼 연결
    ```
    
    ### 3. 결과 (Result)
    
    - `TypeError` 및 `AssertionError`를 모두 해결하고 **학습(Train) 및 검증(Validation) 프로세스를 완주**.
    - **mAP** 및 **NDS** 지표가 정상적으로 산출됨을 확인.
    - Colab 환경에서 Config 파일 수정 실수를 방지하고, **재현 가능한 학습 파이프라인**을 구축함.
