---
layout: article
title: pandas basic 03
tags: [pandas, numpy, datahandling, python]
---


### 01. Subsetting

 - 전체 테이블 중 일부 테이블을 선택하는 방법을 알아봅니다.

```python
# Select a column with slicer
mydf['col1']

# iloc / loc indexer
mydf[5]
mydf[[10, 20]]
mydf.loc[index, column_names]
mydf.iloc[index_nums, column_nums]
mydf.loc[st_row:ed_row, st_col:ed_col]

# iat / at indexer
mydf.iat[row_num, col_num]
mydf.at[row, col]

# Select rows
mydf[mydf[col1] == value]
```

### 02. Boolean Indexing

 - 불린 인덱싱에 대해 알아봅니다.

```python
# A Boolean Series
mys = mydf[col1]
mys > 100
mys.sum()
mys.mean()
mys.value_counts(normalize=True)

# A Boolean Series 2
mys = mydf[col1] > mydf[col2]

# Making criteria
criteria = mydf.col1.isin(['val1'])
mydf[criteria]
mydf.loc[criteria]
mydf.loc[criteria, :]
mydf.iloc[criteria.values]
```

### 03. query & where & mask

 - Pandas Dataframe을 선택하는 세 가지 중요한 메소드를 살펴봅니다.

```python
# query -> SQL문의 where절과 같은 역할을 하며, 문자열 안에 조건을 나열
condition = ['v1', 'v2', 'v3']
qr = "COL1 in @condition and COL2 == 'value' and 10 <= COL3 <= 30"
result = mydf.query(qr)

# where -> df의 shape을 유지하며, 조건에 해당하지 않는 대상의 값을 변경
mydf.where(cond, other=-999)

# mask -> mask 조건에 해당하는 대상의 값을 변경
mydf.mask(criteria).dropna(how='all')
```

### 04. Index Control
 
 - Index는 Series or DataFrame과 많은 부분이 비슷하지만 또 다른 면들도 존재합니다. 그러한 사항들에 대해 살펴봅니다.

 ```python
columns = mydf.columns

 # 문자열 덧셈은 postfix를 생성함
 columns + '_post'

 # 비교식은 Boolean 리스트를 생성
 Columns > 'P'

# Index 생성 후, 바로 값 변경이 불가능함
Columns[1] = 'new_val'

# Index는 합집합, 교집합, 차집합, 대칭차집합 같은 연산이 가능함
columns1.union(columns2)
columns1.symmetric_difference(columns2)
 ```


