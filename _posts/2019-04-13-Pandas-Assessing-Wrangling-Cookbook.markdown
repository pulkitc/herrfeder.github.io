---
layout: post
title: Pandas Cookbook Part 2 (Assessing/Wrangling)
date: 2019-04-13 0:0:0 +0800
categories: datascience
tags: pandas datascience cookbook cheatsheet python
img: "https://pandas.pydata.org/_static/pandas_logo.png" 
---


```python
import pandas as pd
import numpy as np
```

# Assessing


```python
wrangling_df = pd.read_csv("patients.csv")
wrangling_df.shape
```




    (503, 14)




```python
wrangling_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>patient_id</th>
      <th>assigned_sex</th>
      <th>given_name</th>
      <th>surname</th>
      <th>address</th>
      <th>city</th>
      <th>state</th>
      <th>zip_code</th>
      <th>country</th>
      <th>contact</th>
      <th>birthdate</th>
      <th>weight</th>
      <th>height</th>
      <th>bmi</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>female</td>
      <td>Zoe</td>
      <td>Wellish</td>
      <td>576 Brown Bear Drive</td>
      <td>Rancho California</td>
      <td>California</td>
      <td>92390.0</td>
      <td>United States</td>
      <td>951-719-9170ZoeWellish@superrito.com</td>
      <td>7/10/1976</td>
      <td>121.7</td>
      <td>66</td>
      <td>19.6</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>female</td>
      <td>Pamela</td>
      <td>Hill</td>
      <td>2370 University Hill Road</td>
      <td>Armstrong</td>
      <td>Illinois</td>
      <td>61812.0</td>
      <td>United States</td>
      <td>PamelaSHill@cuvox.de+1 (217) 569-3204</td>
      <td>4/3/1967</td>
      <td>118.8</td>
      <td>66</td>
      <td>19.2</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>male</td>
      <td>Jae</td>
      <td>Debord</td>
      <td>1493 Poling Farm Road</td>
      <td>York</td>
      <td>Nebraska</td>
      <td>68467.0</td>
      <td>United States</td>
      <td>402-363-6804JaeMDebord@gustr.com</td>
      <td>2/19/1980</td>
      <td>177.8</td>
      <td>71</td>
      <td>24.8</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>male</td>
      <td>Liêm</td>
      <td>Phan</td>
      <td>2335 Webster Street</td>
      <td>Woodbridge</td>
      <td>NJ</td>
      <td>7095.0</td>
      <td>United States</td>
      <td>PhanBaLiem@jourrapide.com+1 (732) 636-8246</td>
      <td>7/26/1951</td>
      <td>220.9</td>
      <td>70</td>
      <td>31.7</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>male</td>
      <td>Tim</td>
      <td>Neudorf</td>
      <td>1428 Turkey Pen Lane</td>
      <td>Dothan</td>
      <td>AL</td>
      <td>36303.0</td>
      <td>United States</td>
      <td>334-515-7487TimNeudorf@cuvox.de</td>
      <td>2/18/1928</td>
      <td>192.3</td>
      <td>27</td>
      <td>26.1</td>
    </tr>
  </tbody>
</table>
</div>



### Return values for column where value starts with specific string

Using List Comprehension and a classical for loop to go over a DataFrame column:


```python
[row for row in wrangling_df['given_name'] if row.startswith('David')]
```




    ['David', 'David']



Using pandas __built-in__ iterrows will run much faster, as it won't call the complete Pandas Object on every loop iteration:


```python
[row[1][2] for row in wrangling_df.iterrows() if row[1][2].startswith('David')]
```




    ['David', 'David']



### Summarize boolean occurences

By multiplying a boolean with one, we can turn it into a integer and are able to sum up all occurences:


```python
(1*(wrangling_df['given_name'] == "Jake")).sum()
```




    1



### Get Column Keys

Save column names into a list:


```python
wrangling_df.keys()[-3:].tolist()
```




    ['weight', 'height', 'bmi']



### Filtering

Retrieve categorical values based on occurence limit in dataframe:


```python
[x for x,y in list(wrangling_df['height'].value_counts().iteritems()) if y > 30]
# the result will express the heights, that occured more than thirty times in the dataframe
```




    [67, 69, 65, 63, 66, 70, 72, 61]



# Wrangling

## Cleaning
### Duplicates and NaN values

Identify duplicated and null values in address column:


```python
wrangling_df[((wrangling_df.address.duplicated()) & wrangling_df.address.notnull())].head(1)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>patient_id</th>
      <th>assigned_sex</th>
      <th>given_name</th>
      <th>surname</th>
      <th>address</th>
      <th>city</th>
      <th>state</th>
      <th>zip_code</th>
      <th>country</th>
      <th>contact</th>
      <th>birthdate</th>
      <th>weight</th>
      <th>height</th>
      <th>bmi</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>29</th>
      <td>30</td>
      <td>male</td>
      <td>Jake</td>
      <td>Jakobsen</td>
      <td>648 Old Dear Lane</td>
      <td>Port Jervis</td>
      <td>New York</td>
      <td>12771.0</td>
      <td>United States</td>
      <td>JakobCJakobsen@einrot.com+1 (845) 858-7707</td>
      <td>8/1/1985</td>
      <td>155.8</td>
      <td>67</td>
      <td>24.4</td>
    </tr>
  </tbody>
</table>
</div>



Remove these values:


```python
clean_df = wrangling_df[~((wrangling_df.address.duplicated()) & wrangling_df.address.notnull())].head(1)
```

## Remove Columns/Values

### Drop

Drop rows with NaN values depending on subset of columns:


```python
wrangling_df.dropna().shape
```




    (491, 14)




```python
wrangling_df.dropna(subset = ['surname']).shape
```




    (503, 14)



Drop whole columns:


```python
wrangling_df.drop(columns=['contact','birthdate']).shape
```




    (503, 12)



## Create Columns

### Categorical Column

We can create a categorical column that will only allow these values to insert:


```python
wrangling_df['illness'].index
```


    ---------------------------------------------------------------------------

    KeyError                                  Traceback (most recent call last)

    /opt/conda/lib/python3.6/site-packages/pandas/core/indexes/base.py in get_loc(self, key, method, tolerance)
       3077             try:
    -> 3078                 return self._engine.get_loc(key)
       3079             except KeyError:


    pandas/_libs/index.pyx in pandas._libs.index.IndexEngine.get_loc()


    pandas/_libs/index.pyx in pandas._libs.index.IndexEngine.get_loc()


    pandas/_libs/hashtable_class_helper.pxi in pandas._libs.hashtable.PyObjectHashTable.get_item()


    pandas/_libs/hashtable_class_helper.pxi in pandas._libs.hashtable.PyObjectHashTable.get_item()


    KeyError: 'illness'

    
    During handling of the above exception, another exception occurred:


    KeyError                                  Traceback (most recent call last)

    <ipython-input-14-035e5713d5dc> in <module>
    ----> 1 wrangling_df['illness'].index
    

    /opt/conda/lib/python3.6/site-packages/pandas/core/frame.py in __getitem__(self, key)
       2686             return self._getitem_multilevel(key)
       2687         else:
    -> 2688             return self._getitem_column(key)
       2689 
       2690     def _getitem_column(self, key):


    /opt/conda/lib/python3.6/site-packages/pandas/core/frame.py in _getitem_column(self, key)
       2693         # get column
       2694         if self.columns.is_unique:
    -> 2695             return self._get_item_cache(key)
       2696 
       2697         # duplicate columns & possible reduce dimensionality


    /opt/conda/lib/python3.6/site-packages/pandas/core/generic.py in _get_item_cache(self, item)
       2487         res = cache.get(item)
       2488         if res is None:
    -> 2489             values = self._data.get(item)
       2490             res = self._box_item_values(item, values)
       2491             cache[item] = res


    /opt/conda/lib/python3.6/site-packages/pandas/core/internals.py in get(self, item, fastpath)
       4113 
       4114             if not isna(item):
    -> 4115                 loc = self.items.get_loc(item)
       4116             else:
       4117                 indexer = np.arange(len(self.items))[isna(self.items)]


    /opt/conda/lib/python3.6/site-packages/pandas/core/indexes/base.py in get_loc(self, key, method, tolerance)
       3078                 return self._engine.get_loc(key)
       3079             except KeyError:
    -> 3080                 return self._engine.get_loc(self._maybe_cast_indexer(key))
       3081 
       3082         indexer = self.get_indexer([key], method=method, tolerance=tolerance)


    pandas/_libs/index.pyx in pandas._libs.index.IndexEngine.get_loc()


    pandas/_libs/index.pyx in pandas._libs.index.IndexEngine.get_loc()


    pandas/_libs/hashtable_class_helper.pxi in pandas._libs.hashtable.PyObjectHashTable.get_item()


    pandas/_libs/hashtable_class_helper.pxi in pandas._libs.hashtable.PyObjectHashTable.get_item()


    KeyError: 'illness'



```python
wrangling_df['illness'] = pd.Series(pd.Categorical(values=["none"]*len(wrangling_df),categories=["none","a bit","intermediate","very ill"]))
```


```python
wrangling_df['illness'][0] = "bad"
```


    ---------------------------------------------------------------------------

    ValueError                                Traceback (most recent call last)

    <ipython-input-16-6eb1b636cf9e> in <module>
    ----> 1 wrangling_df['illness'][0] = "bad"
    

    /opt/conda/lib/python3.6/site-packages/pandas/core/series.py in __setitem__(self, key, value)
        937         # do the setitem
        938         cacher_needs_updating = self._check_is_chained_assignment_possible()
    --> 939         setitem(key, value)
        940         if cacher_needs_updating:
        941             self._maybe_update_cacher()


    /opt/conda/lib/python3.6/site-packages/pandas/core/series.py in setitem(key, value)
        933                     pass
        934 
    --> 935             self._set_with(key, value)
        936 
        937         # do the setitem


    /opt/conda/lib/python3.6/site-packages/pandas/core/series.py in _set_with(self, key, value)
        975             if key_type == 'integer':
        976                 if self.index.inferred_type == 'integer':
    --> 977                     self._set_labels(key, value)
        978                 else:
        979                     return self._set_values(key, value)


    /opt/conda/lib/python3.6/site-packages/pandas/core/series.py in _set_labels(self, key, value)
        992         if mask.any():
        993             raise ValueError('%s not contained in the index' % str(key[mask]))
    --> 994         self._set_values(indexer, value)
        995 
        996     def _set_values(self, key, value):


    /opt/conda/lib/python3.6/site-packages/pandas/core/series.py in _set_values(self, key, value)
        997         if isinstance(key, Series):
        998             key = key._values
    --> 999         self._data = self._data.setitem(indexer=key, value=value)
       1000         self._maybe_update_cacher()
       1001 


    /opt/conda/lib/python3.6/site-packages/pandas/core/internals.py in setitem(self, **kwargs)
       3691 
       3692     def setitem(self, **kwargs):
    -> 3693         return self.apply('setitem', **kwargs)
       3694 
       3695     def putmask(self, **kwargs):


    /opt/conda/lib/python3.6/site-packages/pandas/core/internals.py in apply(self, f, axes, filter, do_integrity_check, consolidate, **kwargs)
       3579 
       3580             kwargs['mgr'] = self
    -> 3581             applied = getattr(b, f)(**kwargs)
       3582             result_blocks = _extend_blocks(applied, result_blocks)
       3583 


    /opt/conda/lib/python3.6/site-packages/pandas/core/internals.py in setitem(self, indexer, value, mgr)
       1932 
       1933         check_setitem_lengths(indexer, value, self.values)
    -> 1934         self.values[indexer] = value
       1935         return self
       1936 


    /opt/conda/lib/python3.6/site-packages/pandas/core/arrays/categorical.py in __setitem__(self, key, value)
       1979         # something to np.nan
       1980         if len(to_add) and not isna(to_add).all():
    -> 1981             raise ValueError("Cannot setitem on a Categorical with a new "
       1982                              "category, set the categories first")
       1983 


    ValueError: Cannot setitem on a Categorical with a new category, set the categories first



```python
wrangling_df['illness'][0] = "intermediate"
```

## Rename Columns


```python
wrangling_df_2 = wrangling_df.copy()
```

### Different Methods to add suffix to column names

Use __built-in__ pandas functions:


```python
wrangling_df_2 = wrangling_df_2.add_suffix('X')
wrangling_df_2 = wrangling_df_2.add_prefix('X')
wrangling_df_2.head(1)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Xpatient_idX</th>
      <th>Xassigned_sexX</th>
      <th>Xgiven_nameX</th>
      <th>XsurnameX</th>
      <th>XaddressX</th>
      <th>XcityX</th>
      <th>XstateX</th>
      <th>Xzip_codeX</th>
      <th>XcountryX</th>
      <th>XcontactX</th>
      <th>XbirthdateX</th>
      <th>XweightX</th>
      <th>XheightX</th>
      <th>XbmiX</th>
      <th>XillnessX</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>female</td>
      <td>Zoe</td>
      <td>Wellish</td>
      <td>576 Brown Bear Drive</td>
      <td>Rancho California</td>
      <td>California</td>
      <td>92390.0</td>
      <td>United States</td>
      <td>951-719-9170ZoeWellish@superrito.com</td>
      <td>7/10/1976</td>
      <td>121.7</td>
      <td>66</td>
      <td>19.6</td>
      <td>intermediate</td>
    </tr>
  </tbody>
</table>
</div>



Use __rename__ and formatter or lambda for renaming all columns:


```python
wrangling_df_2.rename(columns='{}Y'.format, inplace=True)
wrangling_df_2.rename(columns = lambda x: x+'Z', inplace=True)
wrangling_df_2.head(1)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Xpatient_idXYZ</th>
      <th>Xassigned_sexXYZ</th>
      <th>Xgiven_nameXYZ</th>
      <th>XsurnameXYZ</th>
      <th>XaddressXYZ</th>
      <th>XcityXYZ</th>
      <th>XstateXYZ</th>
      <th>Xzip_codeXYZ</th>
      <th>XcountryXYZ</th>
      <th>XcontactXYZ</th>
      <th>XbirthdateXYZ</th>
      <th>XweightXYZ</th>
      <th>XheightXYZ</th>
      <th>XbmiXYZ</th>
      <th>XillnessXYZ</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>female</td>
      <td>Zoe</td>
      <td>Wellish</td>
      <td>576 Brown Bear Drive</td>
      <td>Rancho California</td>
      <td>California</td>
      <td>92390.0</td>
      <td>United States</td>
      <td>951-719-9170ZoeWellish@superrito.com</td>
      <td>7/10/1976</td>
      <td>121.7</td>
      <td>66</td>
      <td>19.6</td>
      <td>intermediate</td>
    </tr>
  </tbody>
</table>
</div>




```python
wrangling_df_2 = wrangling_df.copy()
```

Use __list comprehensions__ or the __map__ function to do the job:


```python
wrangling_df_2.columns = [column + 'X' for column in wrangling_df_2.columns]
wrangling_df_2.columns = list(map(lambda s: s+'Y', wrangling_df_2.columns))
wrangling_df_2.head(1)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>patient_idXY</th>
      <th>assigned_sexXY</th>
      <th>given_nameXY</th>
      <th>surnameXY</th>
      <th>addressXY</th>
      <th>cityXY</th>
      <th>stateXY</th>
      <th>zip_codeXY</th>
      <th>countryXY</th>
      <th>contactXY</th>
      <th>birthdateXY</th>
      <th>weightXY</th>
      <th>heightXY</th>
      <th>bmiXY</th>
      <th>illnessXY</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>female</td>
      <td>Zoe</td>
      <td>Wellish</td>
      <td>576 Brown Bear Drive</td>
      <td>Rancho California</td>
      <td>California</td>
      <td>92390.0</td>
      <td>United States</td>
      <td>951-719-9170ZoeWellish@superrito.com</td>
      <td>7/10/1976</td>
      <td>121.7</td>
      <td>66</td>
      <td>19.6</td>
      <td>intermediate</td>
    </tr>
  </tbody>
</table>
</div>



## Change Values

### Lamdba/Apply Functions

Using a If-Else function. We have to remember, a lambda function has to generate a return value for every iteration, therefore the else is mandatory:


```python
wrangling_df['height_result'] = wrangling_df['height'].apply(lambda x: "extrem height" if (x > 67) or (x < 60) else "normal height")
wrangling_df.head(5)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>patient_id</th>
      <th>assigned_sex</th>
      <th>given_name</th>
      <th>surname</th>
      <th>address</th>
      <th>city</th>
      <th>state</th>
      <th>zip_code</th>
      <th>country</th>
      <th>contact</th>
      <th>birthdate</th>
      <th>weight</th>
      <th>height</th>
      <th>bmi</th>
      <th>illness</th>
      <th>height_result</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>female</td>
      <td>Zoe</td>
      <td>Wellish</td>
      <td>576 Brown Bear Drive</td>
      <td>Rancho California</td>
      <td>California</td>
      <td>92390.0</td>
      <td>United States</td>
      <td>951-719-9170ZoeWellish@superrito.com</td>
      <td>7/10/1976</td>
      <td>121.7</td>
      <td>66</td>
      <td>19.6</td>
      <td>intermediate</td>
      <td>normal height</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>female</td>
      <td>Pamela</td>
      <td>Hill</td>
      <td>2370 University Hill Road</td>
      <td>Armstrong</td>
      <td>Illinois</td>
      <td>61812.0</td>
      <td>United States</td>
      <td>PamelaSHill@cuvox.de+1 (217) 569-3204</td>
      <td>4/3/1967</td>
      <td>118.8</td>
      <td>66</td>
      <td>19.2</td>
      <td>none</td>
      <td>normal height</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>male</td>
      <td>Jae</td>
      <td>Debord</td>
      <td>1493 Poling Farm Road</td>
      <td>York</td>
      <td>Nebraska</td>
      <td>68467.0</td>
      <td>United States</td>
      <td>402-363-6804JaeMDebord@gustr.com</td>
      <td>2/19/1980</td>
      <td>177.8</td>
      <td>71</td>
      <td>24.8</td>
      <td>none</td>
      <td>extrem height</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>male</td>
      <td>Liêm</td>
      <td>Phan</td>
      <td>2335 Webster Street</td>
      <td>Woodbridge</td>
      <td>NJ</td>
      <td>7095.0</td>
      <td>United States</td>
      <td>PhanBaLiem@jourrapide.com+1 (732) 636-8246</td>
      <td>7/26/1951</td>
      <td>220.9</td>
      <td>70</td>
      <td>31.7</td>
      <td>none</td>
      <td>extrem height</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>male</td>
      <td>Tim</td>
      <td>Neudorf</td>
      <td>1428 Turkey Pen Lane</td>
      <td>Dothan</td>
      <td>AL</td>
      <td>36303.0</td>
      <td>United States</td>
      <td>334-515-7487TimNeudorf@cuvox.de</td>
      <td>2/18/1928</td>
      <td>192.3</td>
      <td>27</td>
      <td>26.1</td>
      <td>none</td>
      <td>extrem height</td>
    </tr>
  </tbody>
</table>
</div>



### Use dict to change values programmatically

For better performance it's always advisable to use a apply function, instead a for loop:


```python
# Mapping from full state name to abbreviation
state_abbrev = {'California': 'CA',
                'New York': 'NY',
                'Illinois': 'IL',
                'Florida': 'FL',
                'Nebraska': 'NE'}

# Function to apply
def abbreviate_state(patient):
    if patient['state'] in state_abbrev.keys():
        abbrev = state_abbrev[patient['state']]
        return abbrev
    else:
        return patient['state']
    
wrangling_df['state'] = wrangling_df.apply(abbreviate_state, axis=1)
wrangling_df.head(5)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>patient_id</th>
      <th>assigned_sex</th>
      <th>given_name</th>
      <th>surname</th>
      <th>address</th>
      <th>city</th>
      <th>state</th>
      <th>zip_code</th>
      <th>country</th>
      <th>contact</th>
      <th>birthdate</th>
      <th>weight</th>
      <th>height</th>
      <th>bmi</th>
      <th>illness</th>
      <th>height_result</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>female</td>
      <td>Zoe</td>
      <td>Wellish</td>
      <td>576 Brown Bear Drive</td>
      <td>Rancho California</td>
      <td>CA</td>
      <td>92390.0</td>
      <td>United States</td>
      <td>951-719-9170ZoeWellish@superrito.com</td>
      <td>7/10/1976</td>
      <td>121.7</td>
      <td>66</td>
      <td>19.6</td>
      <td>intermediate</td>
      <td>normal height</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>female</td>
      <td>Pamela</td>
      <td>Hill</td>
      <td>2370 University Hill Road</td>
      <td>Armstrong</td>
      <td>IL</td>
      <td>61812.0</td>
      <td>United States</td>
      <td>PamelaSHill@cuvox.de+1 (217) 569-3204</td>
      <td>4/3/1967</td>
      <td>118.8</td>
      <td>66</td>
      <td>19.2</td>
      <td>none</td>
      <td>normal height</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>male</td>
      <td>Jae</td>
      <td>Debord</td>
      <td>1493 Poling Farm Road</td>
      <td>York</td>
      <td>NE</td>
      <td>68467.0</td>
      <td>United States</td>
      <td>402-363-6804JaeMDebord@gustr.com</td>
      <td>2/19/1980</td>
      <td>177.8</td>
      <td>71</td>
      <td>24.8</td>
      <td>none</td>
      <td>extrem height</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>male</td>
      <td>Liêm</td>
      <td>Phan</td>
      <td>2335 Webster Street</td>
      <td>Woodbridge</td>
      <td>NJ</td>
      <td>7095.0</td>
      <td>United States</td>
      <td>PhanBaLiem@jourrapide.com+1 (732) 636-8246</td>
      <td>7/26/1951</td>
      <td>220.9</td>
      <td>70</td>
      <td>31.7</td>
      <td>none</td>
      <td>extrem height</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>male</td>
      <td>Tim</td>
      <td>Neudorf</td>
      <td>1428 Turkey Pen Lane</td>
      <td>Dothan</td>
      <td>AL</td>
      <td>36303.0</td>
      <td>United States</td>
      <td>334-515-7487TimNeudorf@cuvox.de</td>
      <td>2/18/1928</td>
      <td>192.3</td>
      <td>27</td>
      <td>26.1</td>
      <td>none</td>
      <td>extrem height</td>
    </tr>
  </tbody>
</table>
</div>



### Normalize, Extrapolate, Interpolate

Normalize each row by the sum of each row:


```python
df = pd.DataFrame([[1,2,3],[3,4,5]])
df.div(df.sum(axis=1), axis=0)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0.166667</td>
      <td>0.333333</td>
      <td>0.500000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0.250000</td>
      <td>0.333333</td>
      <td>0.416667</td>
    </tr>
  </tbody>
</table>
</div>



Interpolate automatically using SciPy:


```python
from scipy.interpolate import interp1d

m = interp1d([1,10],[1,100])
int(m(10))
```




    100



Interpolate manually:


```python
ser = pd.Series([1,2,3,4])
100*(ser - ser.min()) / (ser.max() - ser.min())

```




    0      0.000000
    1     33.333333
    2     66.666667
    3    100.000000
    dtype: float64



Extrapolate automatically using SciPy:


```python
m = interp1d([1,100],[1,10],fill_value="extrapolate")
int(m(95))
```




    9



## Insert Values

### Appending, Concat

Append rows to dataframe from noncomplete dict:


```python
app_dict = {'patient_id':wrangling_df['patient_id'].max()+1,
            'given_name':'testuser',
            'surname':'blahblah'}

new_row = pd.DataFrame([app_dict],columns=app_dict.keys())

pd.concat([wrangling_df,new_row], sort=True).reset_index().tail(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>index</th>
      <th>address</th>
      <th>assigned_sex</th>
      <th>birthdate</th>
      <th>bmi</th>
      <th>city</th>
      <th>contact</th>
      <th>country</th>
      <th>given_name</th>
      <th>height</th>
      <th>height_result</th>
      <th>illness</th>
      <th>patient_id</th>
      <th>state</th>
      <th>surname</th>
      <th>weight</th>
      <th>zip_code</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>501</th>
      <td>501</td>
      <td>3652 Boone Crockett Lane</td>
      <td>female</td>
      <td>2/13/1952</td>
      <td>27.7</td>
      <td>Seattle</td>
      <td>ChidaluOnyekaozulu@jourrapide.com1 360 443 2060</td>
      <td>United States</td>
      <td>Chidalu</td>
      <td>67.0</td>
      <td>normal height</td>
      <td>none</td>
      <td>502</td>
      <td>WA</td>
      <td>Onyekaozulu</td>
      <td>176.9</td>
      <td>98109.0</td>
    </tr>
    <tr>
      <th>502</th>
      <td>502</td>
      <td>2778 North Avenue</td>
      <td>male</td>
      <td>5/3/1954</td>
      <td>19.3</td>
      <td>Burr</td>
      <td>PatrickGersten@rhyta.com402-848-4923</td>
      <td>United States</td>
      <td>Pat</td>
      <td>71.0</td>
      <td>extrem height</td>
      <td>none</td>
      <td>503</td>
      <td>NE</td>
      <td>Gersten</td>
      <td>138.2</td>
      <td>68324.0</td>
    </tr>
    <tr>
      <th>503</th>
      <td>0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>testuser</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>504</td>
      <td>NaN</td>
      <td>blahblah</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



### loc

Create masks to insert specific values:


```python
height_mean = wrangling_df.height.mean()
mask = wrangling_df.patient_id == 4509
column_name = 'height'
wrangling_df.loc[mask, column_name] = height_mean * 1.5
```

## Cut and Seperate

### iloc
We can extract multiple columns and rows at once by combining iloc and numpy.r_:


```python
df_cut = wrangling_df.iloc[np.r_[:5,10:15], np.r_[1, 3]]
df_cut.shape
```




    (10, 2)



## Merge, Group And Sum Values

### Summarize categorical values
Find from a list of mutations of all combinations of categorical values where at least 4 are positive:


```python
categories = ["C006_01","C006_02","C006_04","C006_06","C006_07","C006_10"]
all_muts = list(itertools.combinations(categories, 4))

f_s = 'is correct'
result_df = pd.DataFrame()

for m in all_muts:
    temp_df = umfrage_df[(umfrage_df[m[0]] == f_s) & (umfrage_df[m[1]] == f_s) & (umfrage_df[m[2]] == f_s) & (umfrage_df[m[3]] == f_s)]
    result_df = result_df.append(temp_df)
```


    ---------------------------------------------------------------------------

    NameError                                 Traceback (most recent call last)

    <ipython-input-34-77d3e42cd332> in <module>
          1 categories = ["C006_01","C006_02","C006_04","C006_06","C006_07","C006_10"]
    ----> 2 all_muts = list(itertools.combinations(categories, 4))
          3 
          4 f_s = 'is correct'
          5 result_df = pd.DataFrame()


    NameError: name 'itertools' is not defined


Sum multiple categorical questions with the same range into one continuous value:


```python
def merge_categorical(df,col_str,rating_dict):
    
    filter_col = [col for col in df if col.startswith(col_str)]
    subset_df = df[filter_col].fillna(0)
    norm_value = len(filter_col)
    subset_df['sum'] = subset_df.apply(lambda x: (sum([rating_dict[y] for y in x[filter_col]]))/norm_value , axis=1)
    
    return subset_df

rating_dict_freq_lesson = {"Every Lesson":4,"Most Lessons":3,"Some Lessons":2,"Never or Hardly Ever":1,0:0}

teacher_practice = merge_categorical(pisa_df,"ST79Q",rating_dict_freq_lesson)

```


    ---------------------------------------------------------------------------

    NameError                                 Traceback (most recent call last)

    <ipython-input-35-94b1e19f7237> in <module>
         10 rating_dict_freq_lesson = {"Every Lesson":4,"Most Lessons":3,"Some Lessons":2,"Never or Hardly Ever":1,0:0}
         11 
    ---> 12 teacher_practice = merge_categorical(pisa_df,"ST79Q",rating_dict_freq_lesson)
    

    NameError: name 'pisa_df' is not defined


### Merging

Merging two dataframes depending on multiple keys:


```python
wrangling_df.shape
```




    (503, 16)




```python
treatment_df = pd.read_csv("treatments.csv")
wrangling_df.merge(treatment_df,how='outer',left_on=['given_name','surname'],right_on=['given_name','surname']).shape
```




    (783, 21)



It seems that none of the group of keys does match the new Dataframe. Therefore the length of the treatments will be appended and the resulting dataframe will be the sum of both Dataframes.

### Groupby

Create Dataframe that will count occurences grouped by another value. The resulting construct can be converted to DataFrame using unstack:


```python
patients_country = wrangling_df.groupby(by=['country'])['state'].value_counts().sort_index()
patients_country.unstack(1)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>state</th>
      <th>AK</th>
      <th>AL</th>
      <th>AR</th>
      <th>AZ</th>
      <th>CA</th>
      <th>CO</th>
      <th>CT</th>
      <th>DC</th>
      <th>DE</th>
      <th>FL</th>
      <th>...</th>
      <th>SC</th>
      <th>SD</th>
      <th>TN</th>
      <th>TX</th>
      <th>VA</th>
      <th>VT</th>
      <th>WA</th>
      <th>WI</th>
      <th>WV</th>
      <th>WY</th>
    </tr>
    <tr>
      <th>country</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>United States</th>
      <td>1</td>
      <td>9</td>
      <td>4</td>
      <td>4</td>
      <td>60</td>
      <td>4</td>
      <td>5</td>
      <td>2</td>
      <td>3</td>
      <td>22</td>
      <td>...</td>
      <td>5</td>
      <td>3</td>
      <td>9</td>
      <td>32</td>
      <td>11</td>
      <td>2</td>
      <td>8</td>
      <td>10</td>
      <td>3</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
<p>1 rows × 49 columns</p>
</div>



## Regex Use-Cases
### Extract

Often multiple interesting values are concatenated in one string column. We can use Regex to extract them:


```python
wrangling_df['phone_number'] = wrangling_df.contact.str.extract('((?:\+\d{1,2}\s)?\(?\d{3}\)?[\s.-]?\d{3}[\s.-]?\d{4})', expand=True)
wrangling_df['email'] = wrangling_df.contact.str.extract('([a-zA-Z][a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.][a-zA-Z]+)', expand=True)
```

### Find and Replace

Reverse every lowercase alphabetic word:


```python
repl = lambda m: m.group(0)[::-1]
pd.Series(['foo 123', 'bar baz', np.nan]).str.replace(r'[a-z]+', repl)
```




    0    oof 123
    1    rab zab
    2        NaN
    dtype: object



Use Regex Groups:


```python
pat = r"(?P<one>\w+) (?P<two>\w+) (?P<three>\w+)"
repl = lambda m: m.group('one').upper()
pd.Series(['one two three', 'foo bar baz']).str.replace(pat, repl)
```




    0    ONE
    1    FOO
    dtype: object



Return function to return clean values for every Regex outcome:


```python
import re
reg_df = pd.DataFrame(["blah","blubb","blach","blhab","asdblaheh"], columns=['test'])

def regex_filter(val):
    if val:
        mo = re.search("blah",val)
        if mo:
            return True
        else:
            return False
    else:
        return False

reg_df[reg_df['test'].apply(regex_filter)]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>test</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>blah</td>
    </tr>
    <tr>
      <th>4</th>
      <td>asdblaheh</td>
    </tr>
  </tbody>
</table>
</div>



Change Phone Number Format with Replace and Padding (Before):


```python
wrangling_df.phone_number[:5]
```




    0         951-719-9170
    1    +1 (217) 569-3204
    2         402-363-6804
    3    +1 (732) 636-8246
    4         334-515-7487
    Name: phone_number, dtype: object



(After):


```python
wrangling_df.phone_number.str.replace(r'\D+', '').str.pad(11, fillchar='1')[:5]
```




    0    19517199170
    1    12175693204
    2    14023636804
    3    17326368246
    4    13345157487
    Name: phone_number, dtype: object



Change Zip Code Format using Replace and Padding (Before):


```python
wrangling_df.zip_code[:5]
```




    0    92390.0
    1    61812.0
    2    68467.0
    3     7095.0
    4    36303.0
    Name: zip_code, dtype: float64




```python
wrangling_df.zip_code.astype(str).\
str[:-2].\
str.pad(5, fillchar='0').\
replace('0000n', np.nan)[:5] 
```




    0    92390
    1    61812
    2    68467
    3    07095
    4    36303
    Name: zip_code, dtype: object


