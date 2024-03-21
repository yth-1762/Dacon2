# 데이콘 AI 경진대회1

# 일시
- 2023-12-11 ~ 2024-01-02

# 주제
- 서울시 평균 기온 예측 AI 해커톤

# 배경 및 목적
- 1960년부터 2022년의 서울시 기후 데이터를 이용하여 2023년의 평균 기온 예측
 

# 데이터
- https://dacon.io/competitions/official/236179/data(데이터 출처) 
- 데이터 개수 : 10000개
  
| Column | Description |
|--------|-------------|
| user_id | 사용자의 고유 식별자 |
| subscription_duration | 사용자가 서비스에 가입한 기간 (월) |
| recent_login_time | 사용자가 마지막으로 로그인한 시간 (일) |
| average_login_time | 사용자의 일반적인 로그인 시간 |
| average_time_per_learning_session | 각 학습 세션에 소요된 평균 시간 (분) |
| monthly_active_learning_days | 월간 활동적인 학습 일수 |
| total_completed_courses | 완료한 총 코스 수 |
| recent_learning_achievement | 최근 학습 성취도 |
| abandoned_learning_sessions | 중단된 학습 세션 수 |
| community_engagement_level | 커뮤니티 참여도 |
| preferred_difficulty_level | 선호하는 난이도 |
| subscription_type | 구독 유형 |
| customer_inquiry_history | 고객 문의 이력 |
| payment_pattern | 사용자의 지난 3개월 간의 결제 패턴을 10진수로 표현한 값. |
| target | 사용자가 다음 달에도 구독을 계속할지 (1) 또는 취소할지 (0)를 나타냅니다. |

  

# 사용언어/ 최종 선정 모델
- python/ catboost

# 모델 성능 지표
- macrof1

# EDA
- target 변수 비율확인(1: 61.99% / 0: 38.01%)
- 모든 범주형 변수 범주별 비율 확인(preferred_difficulty_level, subscription_type, payment_pattern)
- 수치형 변수(subscription_duration, recent_login_time, average_login_time, average_time_per_learning_session, monthly_active_learning_days, total_completed_courses, recent_learning_achievement, abandoned_learning_sessions, community_engagement_level, customer_inquiry_history)  target 범주별 분포 히스토그램으로 확인
- 수치형 변수별 preferred_difficulty_level count를 target별로 히스토그램, boxplot으로 확인(결과: target이 0인 데이터는 preferred_difficulty_level이 low, high, medium순으로 많고 target이 1인 데이터는 medium, low, high 순으로 많다.)
- 수치형 변수별 subscription_type를 target별로 히스토그램, boxplot으로 확인(결과: target0, target1 모두 subscription_type이 basic이 premium보다 많다.-> 아무래도 어떤 조건이든 basic인 이용자들이 압도적으로 많아서라고 판단)
- 수치형 변수별 payment_pattern를 target별로 히스토그램, boxplot으로 확인(결과: target 0,1 모두 0,1,2,3,4,5,6,7 순으로 count가 많음)
- 수치형 변수의 상관관계를 heatmap으로 파악(가장 높은 것이 약 0.08로 모든 수치형변수 높지 않은 상관관계를 보임)
- 범주형 변수('preferred_difficulty_level', 'payment_pattern)별 subscription_type을 target별로 히스토그램으로 확인(결과: target 0,1 모두 basic이 premium보다 높게 나옴.)

  

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
