---
title: 하이퍼파라미터 튜닝의 모든것
description: 하이퍼파라미터 알잘딱으로 조정하기
categories:
- hyperparameter
- ML
tags: 
- hyperparameter tuning
- optuna
- gridsearchcv
---

# 하이퍼파라미터 튜닝하기
캐글의 Categorical Feature Encoding Challenge에서 범주형 피처들을 다루는 법을 공부하고 있었다.

어느정도 피처 엔지니어링을 끝마치고 머신러닝 알고리즘으로 학습시켜 리더보드에 제출하다보니 score에 욕심이 생겼다.

그래서 하이퍼파라미터를 조정하기 위해서 로지스틱회귀가 학습하는 방법에 대해 다시 복습하고, 각 하이퍼파라미터가 어떤 의미를 내포하고 있는지 간략하게 알아봤다.

>Logistic Regression Hypterparameter

그리고 이것들을 기반으로 C와 class_weight을 중점으로 GridSeachCV를 사용했는데, 초심자라 잘 모르다보니 범위 지정도 어렵고, 성능이 점점 안 좋아지는 것을 느꼈다ㅠ.. RandomSearchCV를 써도 적정 범위를 잘 모르니 마찬가지였다.

그래서 내가 택한 방법은 로그 스케일로 C 값에 따른 score를 확인해보며 범위를 찾긴 했는데... 그래도 성능 향상이 쉽진 않았다.

이게 시간이 좀 적게 걸리면 모를까 시간이 엄청 많이 걸린다ㅠㅠ 그래서 GridSeachCV의 실행시간을 계산하는 방법도 찾아보며 나름 효율적으로 찾아보자 했지만, 결국 시간이 오래 걸린다는 사실은 변하지 않았다.

그래서 더 찾고 찾다가 알게 된 사실!! Optuna를 쓰면 조금 더 효율적으로 찾을 수 있다는 것이다!!!!

## Optuna가 뭔디??
Optuna도 GridSearchCV, RandomSearchCV 처럼 최적의 하이퍼파라미터를 찾아주는건데 Grid, Random은 찾기 위해 결과가 좋든 안 좋든 범위 내의 모든 조합을 다 수행해보고 최적의 결과를 낸다면, Optuna는 하이퍼 파라미터의 조합이 몇 번의 반복 후에 훈련할 가치가 있는지 여부를 결정하고, 개선이 효과적이지 못한 경우 하이퍼 파라미터의 조합의 학습 과정을 중단시키며 찾는다!! 

쉽게 말해 성능 향상이 별로 안 되는 조합은 버려가며 찾기 때문에 찾는 속도가 훨씬 더 빠르다는 것이다ㅎㅎㅎ 그럼 찾기 위한 경우의 수를 더 많이 시도해볼 수 있으니 더 좋은 파라미터를 찾을 수 있지 않을까 생각한다!!

## Optuna 사용법
>optuna의 프로세스
1. 하이퍼파라미터를 탐색할 함수를 만든다.
    - 조정할 파라미터를 dict에 담는다.
        - 하이퍼 파라미터 목록은 공식문서를 참조한다.
            - 정수: trial.suggest_int('하이퍼파라미터명', 시작, 끝)
            - 실수: trial.suggest_float('하이퍼파라미터명', 시작, 끝)
            - 문자: trial.suggest_categorical('하이퍼파라미터명', [종류1, 종류2])
    - 학습할 모델 객체를 만들어서 학습시킨다.
    - 좋은 파라미터를 썼는지 확인하기 위해 score를 정의한다.
2. study 객체를 만들어서 학습 방향, 샘플러를 넘겨준다.
3. 하이퍼파라미터를 탐색할 함수를 통해 최적화를 한다.

사용법을 알았으니 코드를 살펴보자!

```python
# optuna 관련 패키지 불러오기
import optuna 
from optuna import Trial 
from optuna.samplers import TPESampler 

# 로지스틱 회귀 모델, 교차검증 평가지표 불러오기
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import cross_val_score

# 하이퍼파라미터를 탐색할 함수
def objectiveLR(trial: Trial, X_train, y_train):
    # 조정할 하이퍼 파라미터
    params = {
        "C": trial.suggest_float('C', 8e-3, 0.1),
        "max_iter": 1000,
        "class_weight": {0:1, 1:trial.suggest_float('class_weight', 1,1.5)},
        "solver": trial.suggest_categorical('solver', ['liblinear']),
        "random_state": 42,
    }
    # Logistic Regression 모델 객체 생성, parameter 넣어주기
    model=LogisticRegression(**params)

    # 교차검증 수행, 평가지표는 ROC AUC, 5번 수행해서 그 평균을 점수로 반영
    score=cross_val_score(model, X_train, y_train, cv=5, scoring='roc_auc').mean()

    # 점수를 반환해서 피드백하게 함
    return score
```

```python
# study 객체 생성
# direction: score 값을 최대 또는 최소로 하는 방향으로 지정, RMSE는 score가 낮을 수록 좋은 성능을 나타내기 때문에 이를 정할 수 있게 한듯
study = optuna.create_study(direction='maximize', sampler=TPESampler())

# 옵티마이저 생성
# n_trials: 시도 횟수 (미 입력시 Key interrupt가 있을 때까지 무한 반복)
study.optimize(lambda trial:objectiveLR(trial, train_ohe, target), n_trials=50)

# 결과 확인
print(f'Best trial: score{study.best_trial.value}\nBest params: {study.best_trial.params}')
```

이런식으로 사용하면 된다!!

그래서 나는 우선 범위 설정을 위해 로그 스케일로 값을 넣어서 C 값과 class_weight 값에 따라 증가/감소를 확인해서

범위를 대충 정하고 optuna를 계속 돌렸다!!!!!

![캡처](https://user-images.githubusercontent.com/77676907/191870454-f40ddb0a-a077-4027-914e-1089f5329708.PNG)

결과는 1등보다 0.00001점 높다ㅎㅎ 임의로 주어진 데이터고 어떻게 보면 정말 작은 점수가 올라가서 많이 유의미하진 않지만

영끌 파라미터 조정을 해볼 수 있는 경험이어서 좋았다!