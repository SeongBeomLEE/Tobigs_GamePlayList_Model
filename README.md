# 겜플리 (이미지 · 텍스트 기반 나만의 추천 게임플레이리스트) - 개인 정리
# 소개
- 본 프로젝트는 빅데이터 연합동아리 투빅스 제 12회 데이터 분석 컨퍼런스에서 발표한 프로젝트입니다. 
- 저는 본 프로젝트에서 데이터 수집 및 전처리, 모델링을 맡았습니다.
- Steam 사이트 속 약 3만 명의 유저 정보 데이터(유저의 리뷰, 플레이한 게임 목록, 플레이 시간 등)와 약 2만 5천 개의 게임 정보 데이터(게임 타이틀 이미지, 장르, 게임 설명 등)를 수집하였고, 이를 활용하여 Game2Vec을 구현하고, Game2Vec이 좋은 Representation을 가졌는지 평가하기 위해서 GMF, NCF, NMF, DCN, DeepFM 등의 모델을 구현했습니다.
- 최종적으로 Game2Vec을 활용하여 유사한 게임을 추천해주는 웹사이트를 제작한 게임 추천 프로젝트입니다.

# 요약
- **사용자가 선택한 게임과 유사한 게임을 추천해주는 것이 목표**
- **Data (Pandas, Numpy, Beautifulsoup)**
    - Steam 사이트 속 약 3만 명의 유저 정보, 약 2만 5천 개의 게임 정보 데이터 수집
    - 각 모델의 input 형태에 맞게 데이터 전처리(Sequence, Image, one-hot 등)
- **Game2Vec 생성 Model (Pytorch)**
    - Prod2Vec
        - 유저의 게임 플레이 Sequence를 Window 단위로 학습하여 유저의 게임 선호도 반영
    - Convolutional AutoEncoder
        - 게임의 타이틀 이미지 정보를 압축하는 Embedding을 생성하여 이미지 선호도 반영
    - AutoEncoder
        - 게임의 장르 정보를 압축하는 Embedding을 생성하여 장르 선호도 반영
    - Game2Vec
        - 정규화한 각 Embedding을 concat해 선호도, 이미지, 장르를 고려하는 Embeddding 생성
- **Game2Vec 평가 Model (Pytorch)**
    - GMF, NCF, NMF, DCN, DeepFM 모델 구현
    - 위 모델의 Item Embedding 으로 Gaem2Vec을 활용하여 모델의 성능을 평가
    - Game2Vec을 사용한 모델이 5%P 이상의 성능 향상을 보여줌
- **Deploy**
    - Game2Vec의 cosine similarity 계산해 구한 추천 결과를 저장
    - React와 FastAPI를 이용해 추천 결과 Serving
- [**개인 정리**](https://github.com/SeongBeomLEE/Tobigs_GamePlayList_Model)
- [**Github**](https://github.com/SeongBeomLEE/gameplaylist)
- [**발표 영상**](https://www.youtube.com/watch?v=UpHYyDlUfsQ)
- [**최종 보고서**](http://www.datamarket.kr/xe/board_pdzw77/74633)

# 내용
## 1) **데이터 수집 및 전처리**

- Steam 사이트 속 약 3만 명의 유저 게임 정보 데이터 수집 
(유저의 리뷰, 플레이한 게임 목록, 플레이 시간 등)
- Steam 사이트 속 약 2만 5천 개의 게임 정보 데이터 수집 
(게임 타이틀 이미지, 장르, 게임 설명 등)
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

## 2) **Game2Vec**

- [**Prod2Vec (Sequence)**](https://github.com/SeongBeomLEE/Tobigs_GamePlayList_Model/blob/main/Prod2Vec_(Sequence).ipynb)
    - 유저의 게임 플레이 Sequence를 Window 단위로 학습하여 Game의 Vector를 구함
    - 총 4가지의 경우 중, 유저의 선호도 구분 O / 순서 Shuffle O 의 경우가 가장 높은 성능을 보임
    - 유저의 선호도를 바탕으로 게임을 추천해줌
- [**AutoEncoder (Text)**](https://github.com/SeongBeomLEE/Tobigs_GamePlayList_Model/blob/main/AutoEncoder_(Text).ipynb)
    - 게임의 장르를 AE를 통하여 Encoding하여 Latent Space를 구하여 Game의 Vector를 구함
    - 유사한 장르의 게임을 추천해줌
- [**Convolutional AutoEncoder (Image)**](https://github.com/SeongBeomLEE/Tobigs_GamePlayList_Model/blob/main/Convolutional_AutoEncoder_(Image).ipynb)
    - 게임의 타이틀 이미지를 CAE를 통하여 Encoding하여 Latent Space를 구하여 Game의 Vector를 구함
    - 비슷한 이미지의 게임을 추천해줌
- [**Game2Vec**](https://github.com/SeongBeomLEE/Tobigs_GamePlayList_Model/blob/main/Game2Vec.ipynb)
    - Sequence, Text, Image를 이용하여 구한 각각의 Vector를 Concat하여 Game2Vec을 구함
    - 각 Vector들은 각각 norm 정규화(한쪽의 Vector에 집중되는 편향을 해소시키기 위해)
    - 장르, 선호도, 이미지를 고려하여 게임을 추천 해줌

## 3) **Model**

- Game2Vec이 좋은 Representation을 가졌는지 평가하기 위해서 GMF, NCF, NMF, DCN, DeepFM 모델을 구현함
- Game2Vec을 Collaborative Filtering Model의 Item Embedding 으로 활용
- Item Embedding을 고정 시킨 후 User의 Embedding 을 업데이트하는 방식으로 모델을 학습
- Game2Vec을 사용한 Collaborative Filtering Model의 성능이 Game2Vec을 사용하기 전의 모델보다 성능이 좋다면, Game2Vec은 좋은 Representation을 가졌다고 생각
- 실제로 Game2Vec을 사용한 모델과 사용하지 않은 모델에 성능 차이가 존재했으며, Game2Vec을 사용한 모델이 5%P 이상의 성능 향상을 보여줌
- 따라서 Game2Vec은 Game의 Representation을 잘 나타낸다고 볼 수 있으며, Game2Vec을 이용하여 게임의 cosine similarity 계산해 유사한 게임을 추천해주는 웹사이트를 만듬

| Model | Game2Vec 사용 | ACC | F1-Score | AUC |
| :--- | :--- | :--- | :--- | :--- |
| [General Matrix Factorization](https://github.com/SeongBeomLEE/Tobigs_GamePlayList_Model/blob/main/GMF_Base.ipynb) | X | 78.51% | 0.8711 | 0.7745 |
| [General Matrix Factorization](https://github.com/SeongBeomLEE/Tobigs_GamePlayList_Model/blob/main/GMF.ipynb) | O | 83.62% | 0.8986 | 0.8526 |
| [Neural Collaborative Filtering](https://github.com/SeongBeomLEE/Tobigs_GamePlayList_Model/blob/main/NCF.ipynb) | O | 81.50% | 0.8837 | 0.8391 |
| [Neural Matrix Factorization](https://github.com/SeongBeomLEE/Tobigs_GamePlayList_Model/blob/main/NMF.ipynb) | O | 84.33% | 0.9007 | 0.8715 |
| [Deep & Cross Network Parallel](https://github.com/SeongBeomLEE/Tobigs_GamePlayList_Model/blob/main/DCN_Parallel.ipynb) | O | 82.31% | 0.8868 | 0.8463 |
| [Deep & Cross Network Stacked](https://github.com/SeongBeomLEE/Tobigs_GamePlayList_Model/blob/main/DCN_Stacked.ipynb) | O | 82.55% | 0.8876 | 0.8500 |
| [DeepFM](https://github.com/SeongBeomLEE/Tobigs_GamePlayList_Model/blob/main/DeepFM.ipynb) | O | 83.17% | 0.8929 | 0.8522 |

# 아쉬운 점
- 본인은 본 프로젝트에서 모델링 부분을 맡음, 이에 아직 프론트와 백엔드에 대한 지식이 부족하여 웹 구현에 많은 도움을 주지 못한 것이 아쉬움
- Game2Vec을 만들기 위해 사용했던 모델의 성능을 높이기 위한 작업을 많이 하지 못한 것이 아쉬움
- Game2Vec을 평가하기 위해서 Tool을 구현하여 직접 시각화도 하고 모델도 구현했지만, 조금 더 객관적인 평가 지표가 있으면 좋겠다고 생각함
- 현재 프로젝트는 하나의 게임이 주어졌을 때 그 와 유사한 게임을 추천해주는 방식임
- 본 프로젝트를 조금 더 발전시켜서 2개 이상의 게임이 들어와도 게임을 추천해 줄 수도 있다고 생각됨
- 예를 들어 2개 게임에 Game2Vec의 평균 값을 구하면 2개 게임을 표현할 수 있다고 생각되고, 만들어진 벡터를 이용해 유사한 게임을 찾으면 2개 게임과 매우 유사한 게임을 추천해 줄 수 있다고 생각됨
