
# 4 Bivariate analysis


```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import warnings
warnings.filterwarnings('ignore') # to suppress a numpy warning
%matplotlib inline
```


```python
df = pd.read_csv('train.csv')
```

### Relationship between SalePrice and GrLivArea

When we suspect that the price of a house may depend on another variable, we can visually explore if this seems to be the case by plotting a scatter graph. The X-axis contains one variable, the Y-axis the other variable. Every point is a house from the DataFrame.


```python
df.columns
```




    Index(['Id', 'MSSubClass', 'MSZoning', 'LotFrontage', 'LotArea', 'Street',
           'Alley', 'LotShape', 'LandContour', 'Utilities', 'LotConfig',
           'LandSlope', 'Neighborhood', 'Condition1', 'Condition2', 'BldgType',
           'HouseStyle', 'OverallQual', 'OverallCond', 'YearBuilt', 'YearRemodAdd',
           'RoofStyle', 'RoofMatl', 'Exterior1st', 'Exterior2nd', 'MasVnrType',
           'MasVnrArea', 'ExterQual', 'ExterCond', 'Foundation', 'BsmtQual',
           'BsmtCond', 'BsmtExposure', 'BsmtFinType1', 'BsmtFinSF1',
           'BsmtFinType2', 'BsmtFinSF2', 'BsmtUnfSF', 'TotalBsmtSF', 'Heating',
           'HeatingQC', 'CentralAir', 'Electrical', '1stFlrSF', '2ndFlrSF',
           'LowQualFinSF', 'GrLivArea', 'BsmtFullBath', 'BsmtHalfBath', 'FullBath',
           'HalfBath', 'BedroomAbvGr', 'KitchenAbvGr', 'KitchenQual',
           'TotRmsAbvGrd', 'Functional', 'Fireplaces', 'FireplaceQu', 'GarageType',
           'GarageYrBlt', 'GarageFinish', 'GarageCars', 'GarageArea', 'GarageQual',
           'GarageCond', 'PavedDrive', 'WoodDeckSF', 'OpenPorchSF',
           'EnclosedPorch', '3SsnPorch', 'ScreenPorch', 'PoolArea', 'PoolQC',
           'Fence', 'MiscFeature', 'MiscVal', 'MoSold', 'YrSold', 'SaleType',
           'SaleCondition', 'SalePrice'],
          dtype='object')




```python
#scatter plot grlivarea/saleprice
data = pd.concat([df.SalePrice, df.GrLivArea], axis=1)
data.plot.scatter(x='GrLivArea', y='SalePrice', ylim=(0,800000));
```


![png](output_5_0.png)


Visually, we can see that when we fit a straight line through the data, a line in an upward angle would clearly have the best fit in the sense that the distance between all points and the line would be minimized. The plot indicates a positive linear relationship, if we had to guess the saleprice of a mystery house knowing only the GrLivArea, one best guess would be to read the SalePrice from the fitted line for the given X-value.

### Scatter plot between SalePrice and TotalBsmtSF

We can repeat this process for other variables, for instance is the SalePrice dependent on the size of the basement (TotalBsmtSF appears to be Total Basement in Square Feet)?


```python
data = pd.concat([df.SalePrice, df.TotalBsmtSF], axis=1)
data.plot.scatter(x='TotalBsmtSF', y='SalePrice', ylim=(0,800000));
```


![png](output_8_0.png)


Again, visually it seems a line with an upward angle seems to be a better fit than with a downward angle.

### Associatedness with categorical features


```python
#box plot overallqual/saleprice
data = pd.concat([df.SalePrice, df.GarageType], axis=1)
f, ax = plt.subplots(figsize=(8, 6))
fig = sns.boxplot(x='GarageType', y="SalePrice", data=data)
fig.axis(ymin=0, ymax=800000);
```


![png](output_11_0.png)


It seems that houses with an attached or builtin garage are more expensive.


```python
import matplotlib.pyplot as plt
decades = df.YearBuilt.map(lambda x: x - x % 10)
data = pd.concat([df.SalePrice, decades], axis=1)
f, ax = plt.subplots(figsize=(8, 6))
fig = sns.boxplot(x='YearBuilt', y="SalePrice", data=data)
fig.axis(ymin=0, ymax=800000);
```


![png](output_13_0.png)


It seems that there is no strong correlation here. Some houses before 1900 are expensive, and otherwise more recently build houses tend to be slightly more expensive than older houses.

### Correlation heatmap

Computing these graphs for many pairs of numeric variables can be costly. To quickly explore which variables correlate, we can draw a heatmap over the correlation matrix.


```python
#correlation matrix
corrmat = df.corr()
f, ax = plt.subplots(figsize=(12, 9))
sns.heatmap(corrmat, vmax=.8, square=True);
```


![png](output_16_0.png)


The heatmap shows a darker color red/brown for highly correlated variables. So it seems SalePrice is correlated most strongly with OverallQual and GrLivArea.

Some variables are uncorrelated, yet both appear to have a positive contribution to SalePrice, e.g. FullBath and HalfBath. When we want to learn a prediction, sometimes a joined feature may be a better predictor (not always).


```python
df['Bath'] = df.FullBath + df.HalfBath
```


```python
#correlation matrix
corrmat = df.corr()
f, ax = plt.subplots(figsize=(12, 9))
sns.heatmap(corrmat, vmax=.8, square=True);
```


![png](output_19_0.png)


The combined Bath feature may have a slightly higher correlation, which we can check in the correlation matrix. However, when we check the correlation, it seems Bath is only marginally higher in correlation, and therefore it remains to be seen whether the additional use of Bath (next to FullBath and HalfBath) indeed improves SalePrice predictions. 


```python
corrmat['Bath']['SalePrice']
```




    0.56826651180320198




```python
corrmat['FullBath']['SalePrice']
```




    0.5606637627484452



To make it a bit easier, we can also extract the variables that are most highly correlated with SalePrice, and show a heatmap with the correlations. 


```python
#saleprice correlation matrix
k = 10 #number of variables for heatmap
cols = corrmat.nlargest(k, 'SalePrice')['SalePrice'].index
cm = np.corrcoef(df[cols].values.T)
sns.set(font_scale=1.25)
hm = sns.heatmap(cm, cbar=True, annot=True, square=True, fmt='.2f', annot_kws={'size': 10}, yticklabels=cols.values, xticklabels=cols.values)
plt.show()
```


![png](output_24_0.png)


### Multiple scatter plots

To get more information on how these variables relate to one another, we can use pairplot form the Seaborn library.


```python
sns.set()
cols = ['SalePrice', 'OverallQual', 'GrLivArea', 'GarageCars', 'TotalBsmtSF', 'Bath', 'FullBath', 'YearBuilt']
sns.pairplot(df[cols], size = 2.5)
plt.show();
```


![png](output_26_0.png)


#### Assignment: carry out a bivariate analysis between LotSize and SalePrice. Is there a potential relationship, and what else do you notice?


```python
data = pd.concat([df.SalePrice, df.LotArea], axis=1)
data.plot.scatter(x='LotArea', y='SalePrice', ylim=(0,800000));
data.corr()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>SalePrice</th>
      <th>LotArea</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>SalePrice</th>
      <td>1.000000</td>
      <td>0.263843</td>
    </tr>
    <tr>
      <th>LotArea</th>
      <td>0.263843</td>
      <td>1.000000</td>
    </tr>
  </tbody>
</table>
</div>




![png](output_28_1.png)



```python
from sklearn.cluster import KMeans
kmeans = KMeans(n_clusters=5, random_state=0).fit(data)
centers = kmeans.cluster_centers_
print(centers)
xcenters = []
ycenters = []
for i in centers:
    print("i: " + str(type(i)))
    xcenters.append(i[1])
    ycenters.append(i[0])

print(xcenters, ycenters)
plt.scatter(data.iloc[:,1], data.iloc[:,0], c=kmeans.labels_, cmap="rainbow")
plt.scatter(xcenters, ycenters, marker="D", color="k")
```

    [[ 174321.664        10030.502     ]
     [ 351175.84466019   16161.7184466 ]
     [ 117600.34086956    8338.30782609]
     [ 566346.76923077   20300.07692308]
     [ 244722.29739777   13443.24535316]]
    i: <class 'numpy.ndarray'>
    i: <class 'numpy.ndarray'>
    i: <class 'numpy.ndarray'>
    i: <class 'numpy.ndarray'>
    i: <class 'numpy.ndarray'>
    [10030.502000000004, 16161.718446601961, 8338.3078260869752, 20300.076923076922, 13443.245353159869] [174321.66400000005, 351175.84466019383, 117600.34086956464, 566346.76923076925, 244722.29739776917]





    <matplotlib.collections.PathCollection at 0x7f8b341a4588>




![png](output_29_2.png)

