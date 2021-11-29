# Tobigs_GamePlayList_Model
- Tobigs GamePlatList Project를 진행하면서 만든 추천시스템 모델 코드 및 프로젝트 내용을 정리한 폴더입니다.
- [**전체 코드 및 프로젝트 소개**](https://github.com/SeongBeomLEE/gameplaylist)
- [**발표 영상**](https://www.youtube.com/watch?v=UpHYyDlUfsQ)
- [**최종 보고서**](http://www.datamarket.kr/xe/board_pdzw77/74633)

## 소개
본 프로젝트는 빅데이터 연합동아리 투빅스 제 12회 데이터 분석 컨퍼런스에서 발표한 프로젝트입니다. 
저는 본 프로젝트에서 데이터 수집 및 전처리, 모델링을 맡았습니다. 
Steam 사이트 속 약 3만 명의 유저 정보 데이터(유저의 리뷰, 플레이한 게임 목록, 플레이 시간 등)와 약 2만 5천 개의 게임 정보 데이터(게임 타이틀 이미지, 장르, 게임 설명 등)를 수집하여 게임 추천 프로젝트를 진행했습니다.

## 주제
- 본 프로젝트에서는 Game2Vec을 구현하고, Game2Vec이 좋은 Representation을 가졌는지 평가하기 위해서 GMF, NCF, NMF, DCN, DeepFM 모델을 구현함
- Play Sequence, 이미지, 장르 데이터를 각각 Prod2Vec, Convolutional AutoEncoder, AutoEncoder를 이용하여 Multimodal Feature를 생성함
- Content-Based Model을 통해서 만들어진 Multimodal Feature가 게임들의 Representation을 잘 나타내는 Embedding을 가졌다고 가정하고 이를 Game2Vec이라 지칭함
- Game2Vec을 Collaborative Filtering Model의 Item Embedding 으로 활용함
- Item Embedding을 고정 시킨 후 User의 Embedding 을 업데이트하는 방식으로 모델을 학습함
- Game2Vec을 사용한 Collaborative Filtering Model의 성능이 Game2Vec을 사용하기 전의 모델보다 성능이 좋다면, Game2Vec은 좋은 Representation을 가졌다고 생각함
- 실제로 Game2Vec을 사용한 모델과 사용하지 않은 모델에 성능 차이가 존재했으며, Game2Vec을 사용한 모델이 5% 이상의 성능 향상을 보여줌
- 따라서 Game2Vec은 Game의 Representation을 잘 나타낸다고 볼 수 있으며, 우리는 이러한 Game2Vec을 활용하여 유사한 게임을 추천해주는 웹사이트를 만듬

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
- [Prod2Vec (Sequence)](https://github.com/SeongBeomLEE/Tobigs_GamePlayList_Model/blob/main/Prod2Vec_(Sequence).ipynb)
  - 유저의 게임 플레이 Sequence를 Window 단위로 학습하여 Game의 Vector를 구함
  - 총 4가지의 경우 중, 유저의 선호도 구분 O / 순서 Shuffle O 의 경우가 가장 높은 성능을 보임
  - 유저의 선호도를 바탕으로 게임을 추천해줌
- [AutoEncoder (Text)](https://github.com/SeongBeomLEE/Tobigs_GamePlayList_Model/blob/main/AutoEncoder_(Text).ipynb)
  - 게임의 장르를 AE를 통하여 Encoding하여 Latent Space를 구하여 Game의 Vector를 구함
  - 유사한 장르의 게임을 추천해줌
- [Convolutional AutoEncoder (Image)](https://github.com/SeongBeomLEE/Tobigs_GamePlayList_Model/blob/main/Convolutional_AutoEncoder_(Image).ipynb)
  - 게임의 타이틀 이미지를 CAE를 통하여 Encoding하여 Latent Space를 구하여 Game의 Vector를 구함
  - 비슷한 이미지의 게임을 추천해줌
- [Game2Vec](https://github.com/SeongBeomLEE/Tobigs_GamePlayList_Model/blob/main/Game2Vec.ipynb)
  - Sequence, Text, Image를 이용하여 구한 각각의 Vector를 Concat하여 Game2Vec을 구함
  - 각 Vector를 norm 정규화
  - 장르, 선호도, 이미지를 고려하여 게임을 추천해줌

## Model
| Model | Game2Vec 사용 | ACC | F1-Score | AUC |
| :--- | :--- | :--- | :--- | :--- |
| [General Matrix Factorization](https://github.com/SeongBeomLEE/Tobigs_GamePlayList_Model/blob/main/GMF_Base.ipynb) | X | 78.51% | 0.8711 | 0.7745 |
| [General Matrix Factorization](https://github.com/SeongBeomLEE/Tobigs_GamePlayList_Model/blob/main/GMF.ipynb) | O | 83.62% | 0.8986 | 0.8526 |
| [Neural Collaborative Filtering](https://github.com/SeongBeomLEE/Tobigs_GamePlayList_Model/blob/main/NCF.ipynb) | O | 81.50% | 0.8837 | 0.8391 |
| [Neural Matrix Factorization](https://github.com/SeongBeomLEE/Tobigs_GamePlayList_Model/blob/main/NMF.ipynb) | O | 84.33% | 0.9007 | 0.8715 |
| [Deep & Cross Network Parallel](https://github.com/SeongBeomLEE/Tobigs_GamePlayList_Model/blob/main/DCN_Parallel.ipynb) | O | 82.31% | 0.8868 | 0.8463 |
| [Deep & Cross Network Stacked](https://github.com/SeongBeomLEE/Tobigs_GamePlayList_Model/blob/main/DCN_Stacked.ipynb) | O | 82.55% | 0.8876 | 0.8500 |
| [DeepFM](https://github.com/SeongBeomLEE/Tobigs_GamePlayList_Model/blob/main/DeepFM.ipynb) | O | 83.17% | 0.8929 | 0.8522 |
