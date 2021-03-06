# kakao-arena-brunch-rec

## 모델
- 주어진 태스크에 잘 맞는 **매우매우 심플한 모델**을 구성하였습니다.
- 모델은 **1. candidate 선정** / **2. re-rank** 순서로 이루어져 있습니다.

### 1. candidate 선정 
- 유저의 소비 패턴을 분석해보니 최신 글의 hit ratio 가 매우 높다. => **최신 글 위주로 사용자가 많이 쓸 candidate 를 뽑자**.
   - a. [테스트 시작 몇 일 전 ~ 테스트 기간] 동안 작성된 새로운 문서들 중 **유저가 팔로우한 문서**
   - b. [테스트 시작 몇 일 전 ~ 테스트 기간] 동안 작성된 새로운 문서들 중 **유저가 과거에 많이 본 에디터의 문서**
   - c. 유저가 최근에 본 n 개의 문서 히스토리와 유사한 **article2vec top-k 문서** (모든 기간의 문서)
- a + b + c 를 모두 candidate 문서로 이용.
- 참고) a 만 이용해도 metric 이 눈에 띄게 좋게 잘 나옴.

### 2. re-rank
- **기본적으로 a(중요) > b > c(덜중요) 순서를 유지**하되 
- 문서 리스트 a, b 는 각 리스트를 아래 우선순위에 따라 re-rank 함.
   - 우선순위 1. 유저가 과거에 본 에디터의 글 횟수 -> 많을수록 우선.
   - 우선순위 2. 해당 날짜에 만들어진 문서가 클릭될 확률 순위 -> rank 작을수록 우선.
   - 우선순위 3. 문서가 만들어진 최신순.
- 문서 리스트 c 는 word2vec skip-gram 을 이용해 만든 문서벡터의 cosine similarity 내림차순으로 rank 함.

## Requirements
이 코드는 아래 환경에서 테스트 됨.
- python 3.6
- numpy
- tensorflow 1.13.0 (CPU)
- tqdm

## 파일 구조
- 아래 데이터만 사용.
   - users.json
   - metadata.json
   - predict.tar
   - read.tar
- 데이터 설명 및 다운로드 : https://arena.kakao.com/c/2/data

~~~
├── inference.py
├── util.py
├── config.py
├── word2vec.py
├── train.py
├── /ckpt
└── /res
    ├── users.json
    ├── metadata.json
    ├── predict
    └── read
~~~


## 실행 방법
1. Train word2vec.

   ~~~
   $ python train.py
   ~~~
   
   - 학습 후 모델은 ./ckpt/ 폴더에 저장됨.
   - ~[Deprecated] Download pretrained model (model.ckpt)~

2. Run script.

	~~~
	$ python inference.py
	~~~


3. 아래 user 인풋으로부터 결과 파일을 만듦. **최종 제출 파일은 "./res/predict/recommend.txt"**.
   - ** **test 제출용**
      - user_path: ./res/predict/test.users
      - **output_path: ./res/predict/recommend.txt**


## [UPDATED] Result 
- Overall Rank : 2nd Place (Dinner)
- Final Scores
   - MAP : 0.111363 (1)
   - NDCG : 0.207167 (2)
   - Entropy : 9.8464 (9)
   - Reference : https://arena.kakao.com/forum/topics/168
