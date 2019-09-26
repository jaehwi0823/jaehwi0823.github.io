---
layout: article
title: pandas tablelayout
tags: [pandas, dataframe, datahandling, python, layout]
---

DataFrame의 layout을 원하는 형태로 바꾸는 방법에 대해 살펴봅니다.

### 01. stack & melt & pivot

 - 수평 열 이름을 수직의 관측치로 만들어줍니다.

 ```python
 # 모든 열의 이름을 Index로 변환
 mydf.stack()
 # stack 함수를 원복
 mydf.unstack

# id_vars: 유지, value_vars:전치 *즁요한사실: 기존Index는 삭제됨
mydf.melt(id_vars=['col1'],
          value_vars=['col2', 'col3'],
          var_name='n1',
          value_name='n2')
# melt 함수를 원복
mydf.pivot(index=['col1'],
           columns=['n1'],
           values='n2')
 ```


### 02. wide_to_long

 - 동일한 용어가 반복되는 형태의 열이름은 변환이 쉽습니다.

```python
# prefix로 시작되는 컬럼들을 sep 뒤의 문자로 나눔
# sep 뒤의 문자는 value_column_name이라는 column에 할당됨
pd.wide_to_long(mydf,
                stubnames=['prefix1', 'prefix2'],
                i=['fix_col_name'],
                j='value_column_name',
                sep='_')

pd.wide_to_long(mydf2,
                stubnames=['prefix1', 'prefix2'],
                i=['fix_col_name1', 'fix_col_name2'],
                j='Label',
                suffix='.+', #문자열
                sep='_')
```

### 03. 이름 부여 후 테이블 배치

```python
# column level별 이름을 지정
mydf = mydf.rename_axis(['name1', 'name2'],
                        axis='columns')

# 지정한 이름을 활용하여 swap
mydf.stack('name1').swaplevel('name1', 'n0',
                              axis='columns')
```

### 04. 열 분리

 - 단일 열에 여러가지 정보가 종합된 경우 이를 분리해야 합니다.

```python
# space를 활용해서 단일 열을 분리
newdf = mydf['mycol'].str.split(pat='. ', expand=True)
```
