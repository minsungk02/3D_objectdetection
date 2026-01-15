# 🚗 3D Object Detection with CenterPoint on nuScenes-Mini

![Python](https://img.shields.io/badge/Python-3.10+-blue?logo=python&logoColor=white)
![PyTorch](https://img.shields.io/badge/PyTorch-2.0+-ee4c2c?logo=pytorch&logoColor=white)
![MMDetection3D](https://img.shields.io/badge/MMDetection3D-v1.4-green?logo=openmmlab&logoColor=white)
![Status](https://img.shields.io/badge/Status-Completed-success)

## 📖 프로젝트 개요

본 프로젝트는 **MMDetection3D** 프레임워크를 기반으로 차세대 3D 객체 탐지 알고리즘인 **CenterPoint**를 구현하고 성능을 분석한 연구입니다. 특히 기존 Anchor-based 모델(PointPillars)의 한계를 극복하기 위한 **Anchor-free(Heatmap-based)** 방식의 효율성을 검증하고, **nuScenes v1.0-mini**라는 극히 제한된 데이터 환경에서 모델이 어떻게 공간 정보를 학습하고 일반화하는지를 고찰하는 데 초점을 맞췄습니다.

## 🛠️ 개발 환경 및 기술 스택

* **하드웨어:** Google Colab L4 GPU (VRAM 24GB)
* **프레임워크:** MMDetection3D (OpenMMLab)
* **모델:** CenterPoint (Voxel-based, Anchor-free Head)
* **데이터셋:** nuScenes v1.0-mini (10개 씬)

## 🧠 핵심 모델 아키텍처: Why CenterPoint?

기존 PointPillars와 달리 CenterPoint는 물체를 '상자'가 아닌 **'점(Center)'**으로 인식합니다.

* **Anchor-free:** 미리 정의된 박스(Anchor) 없이 물체의 중심점을 직접 예측하여 3D 공간의 자유도(Degree of Freedom) 문제를 해결합니다.
* **Heatmap-based:** 물체의 존재 확률을 히트맵으로 생성하여, 데이터가 부족한 상황에서도 물체의 위치를 빠르고 정확하게 특정합니다.

## 💥 주요 트러블슈팅: Colab 환경의 의존성 및 검증 이슈

### 1. Colab 환경의 라이브러리 충돌 (spconv, tensorview)

* **현상:** 3D 연산의 핵심인 `spconv` 및 `tensorview` 설치 시 버전 불일치로 인한 빌드 에러 발생.
* **원인:** Colab의 시스템 환경(CUDA/GCC)이 수시로 업데이트되어, 특정 버전에 민감한 3D 라이브러리 간의 의존성 체인이 파손됨.
* **교훈:** 향후 재생산성(Reproducibility)을 위해 **Miniconda**와 같은 패키지 관리자를 활용한 독립적인 가상환경 구축의 중요성을 체득함.

### 2. In-training Validation 실패 및 Bypass 전략

* **현상:** `v1.0-mini` 데이터셋과 `nuscenes-devkit` 간의 샘플 개수 불일치로 학습 중 검증 단계에서 `AssertionError` 발생.
* **해결:** 학습 중 검증을 Bypass(`val_cfg = None`)하고, 학습 완료 후 저장된 체크포인트를 별도로 로드하여 평가하는 **Offline Validation** 파이프라인 구축.

## 📊 실험 결과 및 비교 분석

### 1. 정량적 지표 (CenterPoint vs PointPillars)

동일한 `mini` 데이터셋에서 학습했을 때, CenterPoint가 모든 지표에서 압도적인 성능 우위를 보였습니다.

| 지표 (Metric) | PointPillars (Baseline) | **CenterPoint (Ours)** | 분석 결과 |
| --- | --- | --- | --- |
| **NDS (종합 점수)** | 0.1066 | **0.1833** | **72% 성능 향상** |
| **mAP (정확도)** | 0.0369 | **0.1622** | **약 4.4배 압승** |
| **mATE (위치 오차)** | 0.8031 m | **0.7246 m** | 중심점 탐지의 정밀도 확인 |
| **mAOE (방향 오차)** | 1.0056 rad | **0.9394 rad** | 방향 학습의 어려움 공통 노출 |

### 2. 결과 해석: 왜 CenterPoint인가?

* **Sparsity 대응:** 라이다 데이터의 희소성(Sparsity) 환경에서 불필요한 Anchor 연산을 줄여 연산 효율성을 극대화함.
* **Accuracy:** 데이터가 적은 `mini` 셋임에도 불구하고, mAP가 3.6%에서 16.2%로 폭발적으로 상승한 것은 Anchor-free 방식의 강력한 일반화 능력을 입증함.

## 🧐 성능 분석 및 한계점

* **과적합(Overfitting) 징후:** 학습 Loss는 0에 가깝게 수렴했으나, 테스트 결과인 NDS는 0.18대에 머무름. 이는 모델이 10개의 Scene을 학습하는 과정에서 데이터 자체를 암기해버린 '데이터 포화' 상태임을 의미함.
* **방향(AOE) 오차의 한계:** 약 53도의 높은 방향 오차는 소규모 데이터셋만으로는 물체의 전/후면 특징을 완벽히 학습하기 어려움을 시사함.

## 🚀 향후 개선 방향 (Roadmap)

1. **IoU Prediction Branch 추가:** 모델이 스스로 예측한 박스의 정확도를 평가하게 하여 NDS/mAP 추가 향상 도모.
2. **보조 학습 (Foreground Segmentation):** 점 단위의 세그멘테이션을 학습 과정에 포함하여 백본(Backbone)의 특징 추출 능력 강화.
3. **데이터 확장:** `nuScenes-Full` 데이터셋으로의 확장을 통해 데이터 암기 문제를 해결하고 실제 SOTA 성능(NDS 0.6 이상) 달성 시도.
