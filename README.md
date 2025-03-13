# SW중심대학 디지털 경진대회_SW와 생성AI의 만남 : AI 부문

팀원 | 윤태웅, 이소향, 우건희, 윤수영, 박혜인 (지도교수님 : 김준화 교수님)
## 문제 인식
안녕하세요. 지난 SW 중심 대학 디지털 경진대회 AI 부문의 주제는 생성 AI의 가짜(Fake) 음성 검출 및 탐지 입니다. 평가 지표는 AUC, Brier, ECE로 구성되어있습니다. 고려할 점은 아래와 같습니다.
- 음성 분류 : 공식적인 문서로 확인한 데이터는 [1,1][1,0][0,1][0,0]로 구분되어있다. 
- 노이즈 추가 : test나 train 데이터는 소음 공간이 없다시피 했지만, 사실 들어보니 잡음이 존재
- 성능의 정량화 필요 : 성능 향상을 위한 저오학한 지표 x, 제출한 score 점수 비교가 중요, 성능 향상을 위해서 필수적임 -> 여러가지 방법 비교하면서 Score비교

## 개발 환경
Spyder (로컬) Vscode (서버) 사용
- 로컬 : RTX 3070ti 12GB -> model complexity 고려하여 VRam 8GB 이상 필요
- 라이브러리 설치 및 버전 확인

|이름|버전|
|------|---|
|pandas|1.5.3|
|Numpy|1.25.2|
|Torch|2.2.1+cu121|
|Transformers|4.38.2|
|Sklearn|1.2.2|

라이브러리 import, seed rhwjd : Private Score 재현을 위해 가능한 많은 seed 고정

## 데이터 셋
https://dacon.io/competitions/official/236253/data
- Train Set
  - 55438개의 학습 가능한 32kHz 로 샘플링 된 오디오(ogg) 샘플
  - 방음 환경에서 녹음된 진짜 사람 목소리 샘플과 방음 환경을 가정한 가짜 생성 목소리 (Fake)샘플
  - 각 샘플 당 한명의 진짜 혹은 가짜 목소리가 존재
- Test Set
  - 50000개의 5초 분량의 32kHz로 샘플링 된 평가용 오디오(ogg)샘플
  - TEST_00000.ogg ~ TEST_49999.ogg
  - 방음 환경 혹은 방음 환경이 아닌 환경 모두 존재하며, 각 샘플 당 최대 2명의 목소리(진짜 혹은 가짜)가 존재
- Unlabeled_data
  - 1264개의 5초 분량의 학습 가능한 32kHz 로 샘플링 된 Unlabeled 오디오(ogg) 샘플
  - 평가용 오디오(ogg) 샘플과 동일한 환경이지만 Label은 제공되지 않음

### Train/Valid Dataset Split
train code로 생성한 dataframe(df) 8:2로 분리
분리한 df에서 각 real files 와ㅏ fake files의 존재 확률을 모두 인덱싱해 새로운 df 생성

Train Data : 40,000
Valid Data : 10,000

### Test Noise 제거
Noise가 포함된 Test Data를 DeepFilterNEt(V3)을 이용하여 Noise 제거
https://github.com/Rikorose/DeepFilterNet
![Dacon+SW중심대학+그림1](https://github.com/user-attachments/assets/67de962f-d831-4b62-892e-370af23226a8)

깃허브 사이트에서 제공한 가중치를 사용하여 노이즈 제거 (추가적인 DeepFilterNet 학습 x)

### 데이터 전처리
Data Mixup
|이름|버전|
|------|---|
|R 1|[0,1]|
|F 1|[1,0]|
|R 1 & F 1|[1,1]|
|R 2|[0,1]|
|F 2|[1,0]|
|0|[0,0]|
|0(Gaussian filter 적용)|[0,0]|

데이터 증강 : add noise, time mask
데이터 class의 믹스업을 random.choice에 인덱싱으로 빠짐없이 처리

Custom Dataset (train, val) / Custom Test Dataset (test) Test는 다른 데이터셋 형식 적용

## 모델링
### 전처리 (데이터 증강)
add_noise : random.random() 사용해서 랜덤하게 조건에 맞으면 노이즈 추가
Time mask 추가 : random.random() 사용해서 조건에 만족하면 X축 time(s)축 제거

<img width="876" alt="image" src="https://github.com/user-attachments/assets/39d4a737-3258-4d4b-9217-ac92f0d1e43d" />

### CNNLSTMDeep Model 설계
<img width="646" alt="image" src="https://github.com/user-attachments/assets/f9d94b17-4a1f-47e2-9050-8be4890db3f5" />

### CNNLSTMwithBatchNorm Model 설계
<img width="646" alt="image" src="https://github.com/user-attachments/assets/66b09dcc-0ca6-4df1-8691-0c8037f1dc63" />

### Abalation Study

![Dacon+SW중심대학+그림3](https://github.com/user-attachments/assets/7ca242a7-a861-4005-9a3a-39bd6aa7efd6)

### Experiment
제출할 모든 csv 파일에 sigmoid(k=1) 적용

# 결과

Private Score = 0.5 x (1-AUC) + 0.25 x Brier + 0.25 x ECE
Model 1 = 0.2121
Model 2 = 0.2084
최종 Ensemble 모델 0.2054

# 적용분야
- 딥페이크를 탐지하고 방어할 수 있는 기술을 개발
- 딥페이크 탐지 알고리즘으로 허위 콘텐츠를 신속하게 식별 가능

# 대회 하면서 느낀 점 

딥페이크 기술 탐지를 통해 음성 관련 데이터를 다룬 것은 처음이라 처음엔 어려웠었습니다. 저희는 스펙트로그램으로 변환하여 Real or Fake 를 찾고 싶었는데, 에상과는 다르게 음성 데이터를 그대로 사용하여 학습을 진행했을 때 훨씬 좋은 성능을 보였습니다. 스펙트로그램도 총 3가지를 활용하였는데 전부 스펙트로그램이 잘 추출되지 않아 학습에 전반적으로 영향을 끼친 것이었다고 생각했습니다.
아쉬웠던 점 : 오버피팅이 심하게... 났었습니다. 오버피팅을 방지할 수 있는 방법으로 augmentation을 활용하고자 했었고, 다양한 augmentation을 사용해봤지만 nomalization을 사용한 결과는 정말 쓰레기 성능이 도출 되었었습니다. 모델도 바꿔보고, augmentation도 진행해보며 코드를 객체 지향적으로 설계하는 방식에 대해 많이 배울 수 있었던 시간이었습니다. 코드를 거의 외울 정도로 많이 들여다 봤고, 이후 저는 코드를 볼 수 있게 되었다랄가... 코드 난독증이 어느정도 해결된 것 같아 뿌듯한 경험이었습니다. 교수님들께서 기대 많이 해주셨는데 수상 못해서 면목이 없었습니다..ㅎㅎ 내년 SW 중심대학 대회에서는 수상해오겠다는 다짐을 하였습니다. 



