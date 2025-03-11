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
