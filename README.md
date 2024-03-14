# 골연령 예측 프로젝트 / BoneAge Prediction Project
소아/청소년의 왼손 X-ray 사진을 이용해서 뼈 나이를 예측

## 개발 환경
```
python == 3.12.1
tensorflow == 2.15.0
pytorch == 2.1.0
```

## 프로젝트 설명
소아/청소년의 왼손 X-ray 이미지 데이터를 활용하여 골연령을 예측합니다. 왼쪽 손목과 손가락 관절 X-ray를 ROI(Region Of Interest)로 Object Detection 하여 골 연령을 예측합니다.

## 프로젝트 절차
### 데이터 통합 및 전처리
성별, 나이대로 분류되어 있는 이미지와 엑셀 데이터를 통합하고, 전처리 과정을 진행했습니다.
No.값 중에 114_F, 268_F, 185_M가 중복된 값으로 존재했습니다. 114_F, 268_F는 모든 컬럼이 중복되는 데이터로 1개씩 삭제하였습니다. 185_M는 서로 다른 사람의 데이터이므로, 186_M X-ray 이미지 데이터와 정보가 동일한 행의 No.값을 186_M으로 변경하였습니다.

---

### 이미지 전처리
전체 이미지에 대하여 OpenCV 모듈을 활용하여 Resize, Normalize, Denoise, Equalize, Masking 단계를 거쳐 이미지 전처리를 진행했습니다.

![3_F](https://github.com/jinuHH/BoneAge_Project/assets/105624646/6ce5883f-1358-4123-9763-694de37690a2)

---

### 라벨링 처리 / YOLOv5 모델 학습을 통한 ROI 추출
YOLOv5 모델을 통해 객체를 검출하기 위해서는 label 데이터가 필요했습니다. 따라서 LabelImg 프로그램을 사용하여 TW3를 기반으로 LMCP, MMCP, TMCP, CARPAL 4개의 객체를 지정후 라벨링 작업을 진행하였습니다. mAP 측정결과 0.95로 좋은 성능으로 객체를 검출하는 것을 확인하였습니다.

![results](https://github.com/jinuHH/BoneAge_Project/assets/105624646/eeeaf25e-beab-4aba-96d9-260fa551c85a)

![3_F](https://github.com/jinuHH/BoneAge_Project/assets/105624646/3933d221-4197-4c3e-bb3a-4f63a3d22ac3)

---

### 딥러닝 모델 학습 / Attention-Xception
검출된 LMCP, MMCP, TMCP, CARPAL 데이터를 생성한 Attention-Xception 모델에 학습시킵니다. 모델의 정확도 평가는 회귀에 사용하는 지표인 MAE와 MSE를 사용하였습니다.

![Architecture-of-TjNet](https://github.com/jinuHH/BoneAge_Project/assets/105624646/80c89dcc-a319-4c22-b51d-5061fd7ffa3a)

![화면 캡처 2024-02-12 022232](https://github.com/jinuHH/BoneAge_Project/assets/105624646/59db6aa3-c300-4e7d-aaa4-d709de5987e6)

---

### 모델 결과
학습된 모델 평가 결과 mae = 0.589 로 전문의 판단과 예측 모델의 판단 결과의 차이는 약 0.589년 차이를 보였습니다.
기존 데이터의 골연령 단위는 years이므로, 단위를 months로 변경하였습니다. 즉, 전문의의 판단과 약 7.1개월의 차이를 보였습니다.

관련 논문에서는 약 7.7개월의 차이를 보였습니다.

![화면 캡처 2024-02-12 022314](https://github.com/jinuHH/BoneAge_Project/assets/105624646/ad627823-e70e-48c1-a767-4f6085ee49de)
<https://www.researchgate.net/publication/351221264_Intelligent_Bone_Age_Assessment_An_Automated_System_to_Detect_a_Bone_Growth_Problem_Using_Convolutional_Neural_Networks_with_Attention_Mechanism>

---

### 신장 예측 / LMS Height Prediction
질병관리청에서 제공된 '소아 청소년 성장도표' 분석을 통해 예측된 골연령을 활용하여 18세 기준 예상 신장을 도출하였습니다. 성장도표에는 각 연령의 신장과 그에 따른 백분위수가 포함되어 있습니다. 예측을 위해 LMS기법을 적용했습니다. LMS기법은 연령별 백분위수를 정규분포로 변환시키는 L(Box-Cox Power), M(Median), S(Coefficient of Variation, CV) 값을 찾는 과정입니다. 이를 활용하여 평균정규분포(Z) 데이터를 생성하고, 18세 기준 L, M, S 값과 평균정규분포(Z)로 검사자의 18세 예측 신장값을 도출했습니다. 백분위 범위에서 벗어난 신장값이 도출되면 비정상적인 성장으로 판단하여 Abnormal 값으로 판단했습니다.

2007년 소아 청소년 표준성장도표 소책자 :
<https://www.kdca.go.kr/contents.es?mid=a20303030400>

