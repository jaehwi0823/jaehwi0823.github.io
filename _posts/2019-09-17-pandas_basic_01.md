---
layout: article
title: pandas basic 01
tags: [pandas, numpy, data, python]
---

### 1. pandas basic elements

```python
index = myData.index
columns = myData.columns
data = myData.values
```

### 2. Data types
```python
# check all data types
myData.dtypes

# counts them
myData.get_dtype_counts()
```

### 3. Handling a Series

 - Select a column

```python
# choose one
myData['column_name']
myData.column_name
```

 - if you want to treat it as a dataframe,
```python
mySeries.to_frame()
```

 - check frequencies
```python
# total
mySeries.size
mySeries.shape
len(mySeries)

# not Null only
mySeries.count()
mySeries.notnull().sum()

# counts per item
mySeries.value_counts()
mySeries.value_counts(normalize=True)
```

 - Statistics
```python
# summary
mySeries.describe()

# percentile
mySeries.quantile([.1, .2, .3, .5, .8, .9])
```

 - Treat null
```python
# check null
mySeries.notnull().all()
mySeries.isnull().sum()
mySeries.hasnans

# fill it
mySeries.fillna(0)

# or remove it
mySeries.dropna()
```

 change dtype
```python
mySeries.astype(int)
```

### 4. Index

 - set index
```python
myData.set_index('column')

import pandas as pd
myData = pd.read_csv('./data/d.csv', index_col='index_column')
myData = pd.read_csv('./data/d.csv', index_col='index_column', drop=False)
```

 - bring back
```python
myData.reset_index()
```

 - change index
```python
newData = myData.rename(index={'old_idx':'new_idx'},
                        columns={'old_col':'new_col'})
```


### 5. Column insert / delete
```python
# insert 
idx = myData.columns.get_loc('myCol')
myData.insert(loc=idx+1,
              column=newCol,
              value=myData.V1 - myData.V2)

# Delete
myData = myData.drop('myCol', axis=1)
```
