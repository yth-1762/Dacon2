# 데이콘 AI 경진대회2

# 일시
- 2023-12-11 ~ 2024-01-02

# 주제
- 서울시 평균 기온 예측 AI 해커톤

# 배경 및 목적
- 1960년부터 2022년의 서울시 기후 데이터를 이용하여 2023년의 평균 기온 예측
 

# 데이터
- https://dacon.io/competitions/official/236200/data  (데이터 출처)
- 데이터 개수 : 23011개
  
| Column | Non-Null Count | Dtype  |
|--------|----------------|--------|
| 일시     | 23011 non-null   | object |
| 최고기온   | 23008 non-null   | float64|
| 최저기온   | 23008 non-null   | float64|
| 일교차    | 23007 non-null   | float64|
| 강수량    | 9150 non-null    | float64|
| 평균습도   | 23011 non-null   | float64|
| 평균풍속   | 23007 non-null   | float64|
| 일조합    | 22893 non-null   | float64|
| 일사합    | 18149 non-null   | float64|
| 일조율    | 22645 non-null   | float64|
| 평균기온   | 23011 non-null   | float64|

  

# 사용언어/ 최종 선정 모델
- python/ catboost

# 모델 성능 지표
- MAE

# EDA
- 기간에 따른 평균 기온을 PLOT으로 시각화 -> 정상성을 띄는 것을 확인(ADF TEST 결과도 정상성을 띄는 것으로 확인)
- 연도별 평균기온을 PLOT으로 시각화 -> 우상향하는 경향 확인
- 월별,주차별 평균기온을 PLOT으로 시각화 -> 여름에 가장 평균기온이 높고 겨울에 가장 낮은 것을 확인
- 결측치 개수 확인(최고기온, 최저기온: 3개, 강수량: 13861개, 평균풍속: 4개, 일조합: 118개, 일사합: 4862개, 일조율: 366개)
- 히스토그램으로 강수량 분포 확인(좌로 치우친 것을 확인, 0의 개수가 압도적으로 많음)
- 일사합, 일조합, 일조율, 평균습도, 평균풍속을 히스토그램으로 확인(굳이 정규화를 안시켜도 된다고 판단)
- 일교차와 평균습도와의 관계를 산점도로 확인(별다른 관계가 보이지 않음)
- '일교차', '평균기온', '강수량', '평균습도', '평균풍속' pair plot으로 관계 확인(평균기온과 다른 변수들과의 관계를 살펴보았을 때 강수량을 제외하고는 비슷한 양상을 보임.(특별한 관계가 파악이 되지 않음) 강수량은 늘어날수록 평균기온의 분산이 줄어드는 것을 확인 ->강수량이 변수로 쓰일 가능성이 있다 생각함)
- 수치형 변수 boxplot으로 전체적인 분포 확인(강수량이 0인 값이 압도적으로 많아서 이상치가 많게 나옴)
- '일교차', '평균기온', '강수량', '평균습도', '평균풍속' 변수들 상관관계 확인 위해 heatmap으로 확인(오히려 평균기온과 가장 상관관계가 높은 것은 강수량이 아닌 평균습도였다. 그렇지만 산점도를 보았을 때 평균습도와 크게 연관성이 있는것으로는 보이지 않아 변수로 쓰일지는 미지수)
  

# 데이터 전처리
- 선형보간법을 사용하여 최고기온과 최저기온 결측값 처리(채워진 최고기온과 최저기온값으로 일교차 결측값 문제 해결)
- linear regression 모델로 일조율을 독립변수, 일조합을 종속변수로 설정하여 일조합 예측(결측값 처리) 채워진 일조합으로 다시 독립변수를 일조합 종속변수를 일조율로 설정하여 일조율 예측(일조합,일사합도 똑같은 방식으로 결측값 처리)
- 평균 풍속 결측값은 월별 평균 풍속의 중앙값으로 결측값 대체
- 강수량은 fillna함수의 bfill 방식으로 결측값 처리
- datetime 함수를 활용하여 연,월,일 변수 새로 생성
- 연 변수를 활용하여 주기함수 변수 'Day sin' 'Day cos' 새로 생성
- 주차, 계절(봄,여름,가을,겨울) 변수 새로 생성(계절 변수는 원핫인코딩 수행)
- train data의 '년', '월', '일', 'Day sin', 'Day cos', '주차','계절_가을','계절_겨울','계절_봄','계절_여름'독립변수로 일교차를 종속변수로 하여 catboost fitting-> submission data '년', '월', '일', 'Day sin', 'Day cos', '주차','계절_가을','계절_겨울','계절_봄','계절_여름'변수 활용하여 일교차 변수 예측하여 새로 생성(submission data는 일교차 변수가 따로 없어서 새로 생성)
- 위와 똑같은 방식으로 독립변수에 일교차 추가하여 submission data의 일조율 변수 예측하여 새로 생성
- 위와 똑같은 방식으로 독립변수에 일조율 추가하여 submission data의 평균습도 변수 예측하여 새로 생성
- 위와 똑같은 방식으로 독립변수에 평균습도 추가하여 submission data의 강수량 변수 예측하여 새로 생성


# 모델링
- parameter(random_state=2024,n_estimators=1000,learning_rate=0.01,max_depth=10,reg_lambda=3,verbosity=1) 설정후 xgboost 모델로 fitting(submission data MAE: 3.34)
- randomsearchcv 방식으로 하이퍼파라미터 튜닝을 실시하여 나온 Best Parameters로{'bootstrap': True, 'max_depth': 40, 'max_features': 'log2', 'min_samples_leaf': 19, 'min_samples_split': 12, 'n_estimators': 1000} randomforest 모델로 fitting(submission data MAE: 3.22)
- 가중치를 1:3으로 주어 votingregressor(randomforest+catboost) 모델로fitting(submission data MAE: 2.93)
- parameter(random_state=2024,n_estimators=1000,learning_rate=0.01,depth=10,l2_leaf_reg=3,metric_period=1000,verbose=1000) 설정후 catboost 모델로 fitting(submission data MAE: 2.89) -> 가장 좋은 성능을 보여 최종모델로 선정


# 느낀점
- submission data의 날짜 변수밖에 존재하지 않아 기존 train data의 변수들과 똑같은 변수로 맞추기 위해 모델링을 활용하여 새로운 변수를 생성하였던 것이 좋은 결과로 이어졌다고 생각한다. 이번 대회를 통해 feature engineering의 중요성을 확실하게 느끼게 되었고 다음 대회 때는 더 성능을 높일 수 있는 방안을 더 많이 생각해봐야 될 것 같다.
