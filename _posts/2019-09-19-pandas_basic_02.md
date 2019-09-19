### 01. Column Selection

 - there are a few ways to select columns in a DataFrame.

```python
# select by indexing
mydf[[col1, col2, col3]]

# select by dtype
mydf.select_dtypes(include=['int'])

# select by filter
mydf.filter(like='flag')
mydf.filter(regex='\d')
mydf.filter(like='pre_')

# ex1) select columns having NaN
null_cols = (mydf.select_dtypes(['object']).isnull().sum()>0).to_list()
mydf.select_dtypes(['object']).loc[:, null_cols]
```

 - It is useful to group columns and align them
 
 ```python
 col_A = [col_A1, col_A2, col_A3]
 col_B = [col_B1, col_B2]
 col_C = [col_C1, col_C2, col_C3, col_C4]

 mydf = mydf[col_A + col_B + col_C]
 ```

### 02. Basic Calculation

 - Apply basic calculations to all columns
 
 ```python
# ignore NaN
mydf.count()
mydf.min()
mydf.max()
mydf.sum()
mydf.cumsum()

# consider NaN
mydf.sum(skipna=False)

# ex1) check nulls
mydf.isnull().sum()
mydf.isnull().sum().sum()

# basic calculation
mydf + 2019 # works only when num type
mydf == value
mydf1 == mydf2 # doesn't work when NaN exists
mydf.equal(mydf2)
 ```


### 03. Memory Save
```python
# check
mydf.memory_usage(deep=True)

# type change
mydf.col1 = mydf.col1.astype(np.int8)

# to Categorical
mydf.select_dtypes(include=['object']).nunique()
mydf.col2 = mydf.col2.astype('category')
```


### 04. Largest & Smallest
```python
top1000 = mydf.nlargest(1000, 'col1')
target = top1000.nsmallest(10, 'col2')
```
