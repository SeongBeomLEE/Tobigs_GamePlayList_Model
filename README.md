# Tobigs_GamePlayList_Model
투빅스 게임플레이리스트 프로젝트를 진행하면서 만든 추천시스템 모델 코드가 있는 폴더입니다.

## 주제
- Item2Vec 모델과 Matrix Factorization 모델의 결합
- MF의 관점에서 추천 시스템은 User와 Item의 interaction을 함께 학습시킨다. 그러나 기존의 MF와 달리 우리는 Content-Based Model을 통해서 Item의 Latent Space를 먼저 구한 후 User의 Latent Space를 MF를 통해서 학습시키는 방향으로 진행했다. Content-Based Model을 통해서 구해진 Item의 Vector가 게임들의 Representation을 잘 나타내는 Embedding을 가졌다고 가정하고 이를 Game2Vec이라 지칭하였다. 구해진 Game2Vec을 MF의 Item Embedding으로 활용하여 User의 Embedding을 학습시켜 나갔다. 만약 우리가 만든 모델이 유저의 선호도를 잘 파악한다면 Game2Vec은 좋은 Representation을 가졌다고 볼 수 있다. 따라서 우리는 이러한 게임의 Representation을 잘 나타내는 Game2Vec을 활용하여 유사한 게임을 추천해주는 웹을 구현했다.

## 데이터 수집 및 전처리
- Steam 사이트 속 약 3만 명의 유저 게임 정보 데이터 수집 (유저의 리뷰, 플레이한 게임 목록, 플레이 시간 등)
- Steam 사이트 속 약 2만 5천 개의 게임 정보 데이터 수집 (게임 타이틀 이미지, 장르, 게임 설명 등)
- 플레이한 게임의 수가 5개 미만인 유저 삭제
- 추천 게임이 2개 미만 존재하는 유저 삭제
- 데이터를 분석 가능한 형태로의 기본적인 전처리
- Prod2Vec
  - 유저의 게임 플레이 Sequence를 선호 게임과 비선호 게임으로 분리
  - 분리된 Sequence를 Shuffle 하여 시간의 영향력 제거
  - 위의 과정이 제일 좋은 성능을 보임
- AutoEncoder
  - 각 게임의 장르를 One-hot 인코딩
- Convolutional AutoEncode
  - 각 게임의 타이틀 이미지를 (28, 28, 3) 로 전처리
  - 컴퓨팅 파워의 한계로 이미지의 크기를 줄임, 만약 이미지를 더욱 크게 학습 시킬 수 있다면 더 좋은 성능을 보일 것 
- Model
  - 유저의 플레이 게임 Sequence 를 선호와 비선호로 분리
  - Sequence 를 시간 순으로 나열하여 과거의 80%를 Train Data로 활용
  - 20%의 Data 중 유저를 무작위로 shuffle하여 8 : 2로 Val과 Test Data로 활용

## Game2Vec
- [Prod2Vec (Sequence)]()
  - 유저의 게임 플레이 Sequence를 Window 단위로 학습하여 Game의 Vector를 구함
  - 총 4가지의 경우 중, 유저의 선호도 구분 O / 순서 Shuffle O 의 경우가 가장 높은 성능을 보임
  - 유저의 선호도를 바탕으로 게임을 추천해줌
- [AutoEncoder (Text)]()
  - 게임의 장르를 AE를 통하여 Encoding하여 Latent Space를 구하여 Game의 Vector를 구함
  - 유사한 장르의 게임을 추천해줌
- [Convolutional AutoEncoder (Image)]()
  - 게임의 타이틀 이미지를 CAE를 통하여 Encoding하여 Latent Space를 구하여 Game의 Vector를 구함
  - 비슷한 이미지의 게임을 추천해줌
- [Game2Vec]()
  - Sequence, Text, Image를 이용하여 구한 각각의 Vector를 Concat하여 Game2Vec을 구함
  - 각 Vector를 norm 정규화
  - 장르, 선호도, 이미지를 고려하여 게임을 추천해줌

## Model

| Model | Game2Vec 사용 | ACC | F1-Score | AUC |
| :--- | :--- | :--- | :--- | :--- |
| [General Matrix Factorization]() | X | 78.51% | 0.8711 | 0.7745 |
| [General Matrix Factorization]() | O | 83.62% | 0.8986 | 0.8526 |
| [Neural Collaborative Filtering]() | O | 81.5% | 0.8837 | 0.8391 |
| [Neural Matrix Factorization]() | O | 84.33% | 0.9007 | 0.8715 |
| [Deep & Cross Network Parallel]() | O | 82.31% | 0.8868 | 0.8463 |
| [Deep & Cross Network Stacked]() | O | 82.55% | 0.8876 | 0.8500 |
| [DeepFM]() | O |  |  |  |