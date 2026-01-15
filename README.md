# 3D_objectdetection
![Team_Logo](./background.png)

## 🎬 DEMO VIDEO
<table width="100%">
  <tr>
    <td width="50%" align="center">
      <h3>Scene-0916(FRONT_CAM)</h3>
      <br>
      <video src="https://github.com/user-attachments/assets/f03bf441-18a5-4953-aa60-e5d238398851" 
             width="100%" 
             autoplay 
             loop 
             muted 
             playsinline>
      </video>
    </td>
    <td width="50%" align="center">
      <h3>Scene-0916 (BACK_CAM)</h3>
      <br>
      <video src="https://github.com/user-attachments/assets/806fe88d-af65-43e2-b859-4b7c659e3412" 
             width="100%" 
             autoplay 
             loop 
             muted 
             playsinline>
      </video>
    </td>
    
  </tr>
</table>

## 👨‍💻 Team Members & Workspaces

| Team | Branches |
| :---: | :--- |
| **jeongbin** | [📂Go to branch](https://github.com/minsungk02/3D_objectdetection/tree/jeongbin) |
| **jeonseungho** | [📂Go to branch](https://github.com/minsungk02/3D_objectdetection/tree/jeonseungho) |
| **minsung** | [📂Go to branch](https://github.com/minsungk02/3D_objectdetection/tree/minsung) |
| **sumin** | [📂Go to branch](https://github.com/minsungk02/3D_objectdetection/tree/sumin) |
| **yeseo** | [📂Go to branch](https://github.com/minsungk02/3D_objectdetection/tree/yeseo) |

# 프로젝트 계획서

## 1. 프로젝트 개요

<img width="1328" height="946" alt="image" src="https://github.com/user-attachments/assets/63e19629-4494-420c-8fab-948d2ff3c31d" />


### 주제 선정 배경 및 목표

자율주행 시스템에서 주변 객체의 3차원 위치·크기·거리·방향을 정확히 인식하는 기술은 안전성과 직결되는 핵심 요소이다.

특히 전방 차량, 보행자, 자전거 등의 정확한 거리 추정은 충돌 회피, 제동 제어, 경로 계획 등 상위 ADAS 모듈의 신뢰도를 결정한다.

기존 2D object detection 기반 접근은 RGB 이미지 정보만으로는 distance 정보 추정에 한계가 있으며, 원근·조명·가림에 취약하다. 이에 따라 실제 자율주행 시스템에서는 LiDAR 기반 3D object detection이 필수적으로 활용되고 있다.

**본 프로젝트에서는 nuScenes 데이터셋을 활용하여**

**PointPillars, CenterPoint, PointRCNN 등 대표적인 LiDAR 기반 3D object detection 모델을 구현·비교하고,**

**객체의 3D bounding box 시각화 및 실제 거리 예측 정확도를 정량적으로 평가하는 것을 목표로 한다.**

궁극적으로는

- 3D 객체 인식 파이프라인을 end-to-end로 구현하고
- 각 모델의 구조적 차이와 성능 특성을 비교 분석하며
- 자율주행 환경에서 활용 가능한 정확한 거리 인식 시스템의 구현 가능성을 검증한다.

### 프로젝트 목표

본 프로젝트의 구체적인 목표는 다음과 같다.

1. **nuScenes LiDAR 데이터 기반 3D Object Detection 구현**
    - 차량, 보행자, 자전거 등 주요 클래스 검출
2. **3D Bounding Box 예측 및 시각화**
    - 위치(x, y, z), 크기(l, w, h), yaw 각도 포함한 7-DoF bounding box 생성
3. **객체-자차 간 거리 정확도 평가**
    - 예측된 3D 위치를 기반으로 실제 거리 오차 분석
4. **모델별 성능 비교 분석**
    - PointPillars / CenterPoint / PointRCNN 구조 및 성능 차이 분석

---

## 2. 프로젝트 기대효과

### 3D 객체 인식 및 거리 추정 성능 향상

LiDAR 포인트 클라우드를 직접 활용함으로써

2D 영상 기반 방식 대비 정확한 거리·위치 인식이 가능함을 실험적으로 검증할 수 있다.

### 실제 자율주행 데이터 기반 실험 경험 확보

nuScenes는 다양한 도시 환경, 날씨, 시간대 조건을 포함한 대규모 실제 주행 데이터셋으로,

본 프로젝트를 통해 현실적인 자율주행 시나리오에서의 3D 인식 문제를 다룰 수 있다.

### 최신 LiDAR 기반 3D Detection 모델 비교 분석

- Pillar-based (PointPillars)
- Center-based (CenterPoint)
- Proposal-based (PointRCNN)

서로 다른 접근 방식의 모델을 동일 데이터셋에서 비교함으로써

각 구조의 장단점 및 실용성을 명확히 이해할 수 있다.

### 자율주행 인지(Perception) 핵심 역량 강화

포인트 클라우드 전처리 → 네트워크 학습 → 3D box 추론 → 평가까지

자율주행 perception 파이프라인 전반을 직접 구현함으로써

자율주행 AI 엔지니어링 역량을 체계적으로 강화할 수 있다.

---

## 3. 사용 데이터 및 기술 스택

### 데이터셋

- **nuScenes Dataset**
    - LiDAR Point Cloud (32-beam)
    - 3D Bounding Box GT
    - Multi-class annotations
    - Train / Validation split 활용

### 사용 모델

- **PointPillars**
    - Voxelization 기반 경량 3D detection
- **CenterPoint**
    - Object center heatmap 기반 state-of-the-art 모델
- **PointRCNN**
    - Point-wise feature + proposal 기반 2-stage 모델

### 개발 환경

- Python, PyTorch
- OpenPCDet / MMDetection3D
- CUDA 기반 GPU 환경
- Open3D / Mayavi (3D 시각화)

---

## 4. 평가 지표 및 실험 방법

### 성능 평가 지표

- **mAP (3D, BEV)**
- **Translation Error (m)**
- **Scale Error**
- **Orientation Error**
- **Distance Error (L2 distance)**

### 실험 설계

- 동일한 nuScenes validation set 사용
- 동일 클래스 기준 성능 비교
- 거리 구간별(근거리 / 중거리 / 원거리) 성능 분석


- 팀원 역할 분담 (추후에 작성 예정)
    - 김민성: PointPillars모델 학습 및 추론(MMdetection3D 프레임워크 사용)
    - 김예서: CenterPoint 모델 학습 및 추론(MMdetection3D 프레임워크 사용 X)
    - 송수민: PointPillars모델 학습 및 추론(MMdetection3D 프레임워크 사용)
    - 전승호: CenterPoint 모델 학습 및 추론
    - 한정빈: CenterPoint 모델 학습 및 추론

---
## 📅 Project Schedule

프로젝트 전체 진행 일정입니다.

| 진행 단계 | 상세 내용 | 일정 | 비고 |
| :--- | :--- | :--- | :--- |
| **기획** | ✏️ 프로젝트 주제 정하기 | 2026.01.09 | 완료 |
| **기획** | ✏️ 데이터셋 정하고 진행 방향 결정 | 2026.01.12 | 완료 |
| **중간 점검** | 💬 중간 발표 (화요일 3시) | 2026.01.13 | 완료 |
| **개발 및 정리** | 💡 프로젝트 정리 및 결과 공유 | 2026.01.14 ~ 01.15 | 진행 중 |
| **마무리** | 🔥 최종 발표 | 2026.01.16 | 예정 |

<br>

## 🚘 Model Comparison Study

팀원별 담당 3D Object Detection 모델 및 특징 비교입니다.

| 팀원 | 추천 모델 | 방식 | 특징 및 역할 |
| :---: | :--- | :--- | :--- |
| **1번** | **PointPillars** | LiDAR (2D) | **속도 중심.** 가장 가볍고 빨라 실시간성이 좋음 |
| **2번** | **SECOND** | LiDAR (3D) | **Voxel 기준.** Sparse Convolution의 기초가 되는 모델 |
| **3번** | **CenterPoint** | LiDAR (Center) | **SOTA(최신).** 박스 형태가 아닌 '점'으로 물체를 탐지(Anchor-free) |
| **4번** | **PV-RCNN** | LiDAR (Hybrid) | **최고 정확도.** Voxel과 Point 방식을 합친 하이브리드 구조 |
| **5번** | **PointRCNN** | LiDAR (Point) | **Point 기준.** 2D 변환 없이 점 데이터를 직접 처리하여 손실 최소화 |
---
## 통합 프레임워크

| **비교** | **OpenPCDet** | **MMDetection3D** |
| --- | --- | --- |
| **난이도** | 상대적으로 쉬움 (직관적임) | 조금 어려움 (학습 곡선 높음) |
| **주력 센서** | **LiDAR(라이다)** 특화 | LiDAR + **Camera** + Radar |
|  | "라이다 모델 위주로 빠르게 성과 내자!" | "카메라 모델도 해보고 싶고, 확장성이 중요하다!" |


## Baseline 선정

- MMDetection3D+KITTI 3D (예제)
    
    → 1번: 강사님이 올려주신 코드가 PointPillars + MMDetection3D 조합
    
- MMDetection3D (이론 및 로컬 환경 구현)

