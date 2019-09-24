---
layout: article
title: pandas groupby
tags: [pandas, numpy, datahandling, python, groupby]
---

데이터분석시, 특정 조건의 group별 연산이 매우 빈번하게 발생합니다. Pandas DataFrame에서 해당 작업들을 처리하는 방식을 알아봅니다.

### 01. groupby basic

 - 기본적인 데이터 집계 방법을 살펴봅니다.

```python
# DataFrameGroupBy object 생성
grouped = mydf.groupby('mycolumn')

# DataFrameGroupBy object는 Iterable
for name, group in grouped:
    print(name)
    display(group)

# 각 그룹별 head 확인도 가능
grouped.head(3).head(5)

# 집계 방법1 -> {열:연산} dict를 입력
mydf.groupby('mycolumn').agg({'agg_col': 'sum'})

# 집계 방법2 -> 컬럼 선택후 연산자만 전달
mydf.groupby('mycolumn').['agg_col'].agg('sum')
mydf.groupby('mycolumn').['agg_col'].agg(numpy.sum)

# 집계 방법2 -> agg 메서드 없이 바로 연산 함수 활용 가능
mydf.groupby('mycolumn').['agg_col'].sum()
```

### 02. multi columns and aggs

 - 여러 개의 항목들을 활용하여 집계하는 방법에 대해 알아봅니다.

```python
# 2개 columns 그룹핑
mydf.groupby(['col1', 'col2']).['agg_col'].sum()

# 2개 columns 그루핑, 2개 columns 집계
mydf.groupby(['col1', 'col2']).['agg_col1', 'agg_col2'].sum()

# 2개 columns 그루핑, 2개 columns 두 가지 집계
mydf.groupby(['col1', 'col2']).['agg_col1', 'agg_col2'].agg(['sum', 'mean'])

# 2개 columns 그룹, 2개 columns, 다양한 집계
agg_dict = {'agg_col1':['sum', 'mean', 'size'],
            'agg_col2':['sum', 'var']}
mydf.groupby(['col1', 'col2']).agg(agg_dict)
```

### 03. agg 함수 customizing

 - 사용자 정의 agg 함수 활용법을 이해합니다.

```python
# Input: Series, Output: scalar 인 함수를 정의
def my_agg_func_ex(s):
    return s.max()
# 집계 Column명 수정 가능
my_agg_func_ex.__name__ = 'My Aggregation Function'
# 사용자 정의 함수를 agg에 활용
mydf.groupby('mycolumn').['agg_col'].agg(my_agg_func_ex)
```

 - 함수에 추가 인수를 활용하는 것도 가능합니다.
 - 하지만, 여러 개의 함수를 활용할 때는 '클로저'를 활용해야 합니다.

 ```python
 # 추가 인수를 받도록 정의
def my_agg_func_ex(s, p1, p2):
    return s.between(p1, p2).max()
# 집계 Column명 수정 가능
my_agg_func_ex.__name__ = 'My Aggregation Function'
# 사용자 정의 함수를 agg에 활용하며 동시에 인수 전달
mydf.groupby('mycolumn').['agg_col'].agg(my_agg_func_ex, 10, 200)
 ```


### 04. group filtering

 - 데이터셋을 Grouping 한 상태로 filtering이 가능합니다.
 - 일반적인 SQL query에서는 단일 쿼리로 group by 상태에서 filtering이 불가능합니다.

```python
# Input: DataFrame, Output: A Boolean
def my_filtering_func_ex(df, threshold):
    return result>threshold

# filter 적용: Retrun이 True인 Group만 선택 & 나머지 drop
filtered = mydf.groupby('mycol').filter(my_filtering_func_ex, threshold=.5)
```

### 05. 만능 함수 Apply

 - groupby 객체에는 agg, filter, transform, apply 네 가지 함수를 적용할 수 있습니다.
 - 각각 함수는 Input & Output 양식이 상이합니다.
 > 1. agg(Series -> Scalar)
 > 2. filter(DataFrame -> Scalar)
 > 3. transform(DataFrame -> DataFrame)
 > 4. apply(DataFrame -> 모든양식)
 - 모든 종류의 Gropby 객체 처리가 가능하다는 점에서 유연성이 가장 높습니다.

```python
# 사용자함수1: Scalar 반환
def my_apply_func_ex1(df):
    return result_scalar

# 사용자함수2: Series 반환
def my_apply_func_ex2(df):
    return result_series

# groupby 객체에 반영(모두 가능)
mydf.groupby('mycol').apply(my_apply_func_ex1)
mydf.groupby('mycol').apply(my_apply_func_ex2)
```

### 06. Pandas cut 함수

 - 연속형 변수를 범주화 하는 경우도 빈번하게 일어나므로 사용법을 익혀봅니다.

 ```python
 # 범주형 Series로 변환
 bins = [-np.inf, 100, 200, 300, 400, 500, np.inf]
 labels=['Label1','Label2','Label3','Label4','Label5','Label6']
 cuts = pd.cut(mydf['mycol'], bins=bins, labels=labels)

 # 해당 범주를 그룹으로 활용
 mydf.groupby(cuts)
 ```
