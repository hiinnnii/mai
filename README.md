# SW중심대학 디지털 경진대회_SW와 생성AI의 만남 : AI 부문

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

![Dacon+SW중심대학+그림2](https://github.com/user-attachments/assets/9c33a222-ff4a-44a1-8d74-e047ae8f0686)


## 모델링
