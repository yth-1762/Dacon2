# 데이콘 AI 경진대회1

# 일시
- 2023-12-11 ~ 2024-01-02

# 주제
- 서울시 평균 기온 예측 AI 해커톤

# 배경 및 목적
- 1960년부터 2022년의 서울시 기후 데이터를 이용하여 2023년의 평균 기온 예측
 

# 데이터
- https://dacon.io/competitions/official/236200/data(데이터 출처)
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
- 수치형 변수의 조합으로 feature engineering을 실시하여 8개 변수 생성('learning_intensity', 'learning_efficiency', 'achievement_per_session_time', 'learning_session_time_ratio', 'courses_completed_per_learning_days', 'community_engagement_achievement_interaction', 'abandoned_sessions_per_course_completed', 'abandoned_sessions_ratio')
- lower bound와 upper bound 설정을 통해 수치형 변수 이상치 제거
- subscription_type, payment_pattern 원핫 인코딩, preferred_difficult_level 라벨 인코딩 수행
- 수치형 변수에 루트 값을 씌운 변수와 루트 1/3을 씌운 변수를 추가 생성
- 수치형 변수 전체 min max scaler로 표준화 수행(정규분포를 띄도록 만들기 위해)-> 히스토그램으로 분포 확인 후 정규분포와 유사하지 않은 변수는 제거
- 수치형 변수 2차 다항식 변환 변수 추가 생성
- PolynomialFeatures를 사용하여 상호작용 변수들을 새롭게 생성
- standard scaler로 표준화 수행 후 pca 수행
- smote 수행


# 모델링
- 하이퍼파라미터 튜닝을 optuna를 적용하여 xgboost 모델로 fitting(결과: macrof1= 약 0.50)
- stacking 방법(xgboost+lgbm+catboost // 최종모델:logistic regression)을 활용하여 fitting(결과: 0.496)
- votingclassifier(soft방식/xgboost+lgbm+catboost) 모델을 활용하여 fitting(결과: 약 0.50)
- 하이퍼파라미터 튜닝을 optuna를 적용하여 catboost 모델로 fitting(결과: macrof1= 약 0.51) -> 최종모델 선정


# 느낀점
- 아무래도 처음 나가는 경진대회이다 보니 EDA에서도 유의미하지 않은 시각화를 실시했다고 생각한다. 다음부터는 인사이트를 발견할 수 있도록 유의미한 시각화를 실시해야 한다는 걸 느꼈다. 전처리 또한 어느 정도 전처리가 완료 되었음에도 다른 방법으로 다시 표준화를 실시하는 건 잘못된 수행방법이라 생각한다. 아무래도 처음이다보니 여러가지 방법을 적용해 보고 싶어서 너무 무리하여 전처리를 진행했다. 모델링 같은 경우에도 좀 더 다양한 방법으로 시도해봐야 했었다는 아쉬움이 남는다.
