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
1. penalty: 제약조건(규제)를 설정 (default=l2)
    - none: 규제를 지정하지 않음
    - l2: L2 규제를 가함
    - l1: L1 규제를 가함
    - elasticnet: L1과 L2를 적절히 가함
    - solver 함수가 지원하는 규제를 잘 보고 지정해야 함
2. dual: 이중 또는 초기 공식(?) (default=False)
    - solver로 liblinear를 사용하고, l2 규제에 대해서만 사용됨. 
    - n_samples  > n_features일 때 dual=False로 설정하길 권장함.
3. tol: 계산 시 정지 기준에 대한 공차 (default=1e-4)
4. C: cost function, L1 또는 L2 제약조건의 강도를 설정, (default=1)
    - 높을 수록 낮은 강도의 제약조건이 설정되어 계산된 기울기가 데이터들 쪽으로 치우치게 되고,
    - 낮을 수록 높은 강도의 제약조건이 설정되어 0쪽으로 붙잡히게 된다.
5. fit_intercept: 정규화 효과 정도
6. class_weight : 데이터에 직접 가중치를 설정하여 학습의 강도를 다르게 함 (default-None)
7. random_state: 난수 seed 설정 (default=None)
8. solver: 최적화에 사용할 알고리즘 방식 (default-lbfgs)
    - liblinear: L1, L2 제약조건 두 가지를 모두 지원, 크기가 작은 데이터에 적합하다.
    - sag, saga: 확률적경사하강법을 기반으로 하기 때문에 대용량 데이터에 적합
        - sag는 L1 제약조건만을 지원하고, saga는 L1, L2 둘 다 지원함
    - newton-cg, lbfgs: 멀티클래스 분류에 사용하고, 성능은 lbfgs가 더 낫다고 한다. L2 제약조건만 지원함
9. max_iter: 해를 찾아가는 데 있어서 연산을 반복하는 횟수 (무한루프 방지용) (default=100)
    - 대게 전처리가 잘 되어있지 않으면 연산을 반복하는 횟수가 많아져서 max_iter가 모자라다는 에러가 발생하곤 한다고 함.
10. multi_class: 다중 분류의 경우 설정 (default=’auto’)
11. verbose : 동작 과정에 대한 출력 메시지 (default=0)
12. warm_start : 이전 모델을 초기화로 적합하게 사용할 것인지 여부(?) (default=False)
13. n_jobs : 병렬 처리 할 때 사용되는 CPU 코어 수 (default=None)
14. l1_ratio : L1 규제의 비율(Elastic-Net 믹싱 파라미터 경우에만 사용) (default=None)

지이ㄴ이이이ㅣㅣㄴ짜 많다..!!! 이걸 다 외우냐고? 아니지ㅋㅎ
여기서 내가 설정할 하이퍼파라미터는 max_iter(무한루프 방지용), C, solver, class_weight, random_state 이다!

로지스틱 회귀가 회귀선을 기준으로 분류하여 확률을 계산해내기 때문에 규제의 강도를 결정하는 C가 중요하다고 생가했고, class_weight도 학습 시 중요하다고 생각되어서 결정했다. solver는 데이터의 수가 적기 때문에 liblinear를 사용할 예정이고, 이에 따라 penalty는 default인 L2를 그대로 사용하게 될 것이다. 또한, 일정한 결과를 내기 위해 random_state는 42로 둘 것이고, max_iter는 가서 해봐야 알 것 같다..!

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
