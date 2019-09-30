---
layout: article
title: sklearn basic 01
tags: [sklearn, ml, preprocessing, python]
---

sklearn의 기초 사용법에 대해 학습합니다.

### 01. 데이터 나누기
```python
from sklearn.model_selection import train_test_split

# Train / Validation Split을 손쉽게 수행
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.3, random_state=1, stratify=y
)

# 결과 확인
print(np.bincount(y))
print(np.bincount(y_train))
print(np.bincount(y_test))
```

### 02. 단위 표준화 (Scaling)
```python
# scaler를 생성하고, 학습데이터로 학습
from sklearn.preprocessing import StandardScaler
SS = StandardScaler()
SS.fit(X_train)

# 학습된 scaler를 각 데이터에 적용
X_train_s = SS.transform(X_train)
X_test_s = SS.transform(X_test)
```

### 03. 단순 Perceptron 모형 학습
```python
# 모형 학습
from sklearn.linear_model import Perceptron
ppn = Perceptron(max_iter=10, eta0=0.001, tol=1e-4, random_state=1)
ppn.fit(X_train_s, y_train)

# 오분류율 확인 (= 1- Accuracy)
(y_test != y_pred).sum()/len(y_test)

# 정확도 확인
from sklearn.metrics import accuracy_score
accuracy_score(y_test, y_pred)

# 검증 및 확인을 동시에!
ppn.score(X_test_s, y_test)
```


### 04. Logistic Regression 모형 학습
```python
# 로지스틱 회귀 생성
from sklearn.linear_model import LogisticRegression
LR = LogisticRegression(solver='lbfgs')
LR.fit(X_train_s, y_train)

# 결과 검증
LR.score(X_test_s, y_test)

# 각 관측치의 Target 확률
LR.predict_proba(X_test_s)
```



