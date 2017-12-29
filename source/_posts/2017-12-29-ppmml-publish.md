title: ppmml publish today
date: 2017-12-29 19:42:02
tags: machine learning
---

On the last day before the New Year Holiday, ppmml is published.
[ppmml](https://github.com/lgrcyanny/ppmml) is a python library for converting machine learning models to pmml file. ppmml wraps jpmml libraries and provides clean interface.

# What is pmml file?
PMML - “Predictive Model Markup Language”, which is a standard for XML documents which express trained instances of analytic models.
Various platforms adopt pmml as machine learning model standard, including IBM, SAS, Microsoft, Spark, KNIME etd.[pmml-platforms](http://dmg.org/pmml/products.html)

[jpmml](https://github.com/jpmml) has developed pmml model library and supported models of spark, xgboost, tensorflow, sklearn, lightgbm and R. All of these libraries are separated and written in java.
ppmml wraps jpmml libraries and proved a simple and easy-to-use API for pmml files transformation.
0.0.1 version supports sklearn, tensorflow, spark, lightgbm, xgboost and R models. All models supported by jpmml are supported by ppmml. Common machine learning algorithms are supported, such as Decision Tree, Logistic Regression, GBDT, Random Forest, KMeans. However, Deep Learning support is not ready.

<!--more-->

# Installation
```shell
pip install --default-timeout=10000 -i https://pypi.anaconda.org/lgrcyanny/simple ppmml
```
[ppmml conda package](https://anaconda.org/lgrcyanny/ppmml)

# Geting Started
```python
import pandas as pd
from sklearn.datasets import load_iris
from sklearn.linear_model import LogisticRegression
from sklearn.externals import joblib
import ppmml
# load data and train iris datasets
(X, y) = load_iris(True)
lr = LogisticRegression(tol=1e-5)
lr.fit(X, y)
joblib.dump(lr, "lr.pkl.z", compress = 9)

# to pmml file
ppmml.to_pmml("lr.pkl.z", "lr.pmml", model_type='sklearn')

# prepare test data
df = pd.DataFrame(X)
df.columns = ['x1', 'x2', 'x3', 'x4']
df.to_csv('test.csv', header=True, index=False)
# predit with pmml file, a simple predict API based on jpmml-evaluator
ppmml.predict('lr.pmml', 'test.csv', 'predict.csv')
```

[ppmml github](https://github.com/lgrcyanny/ppmml)

---

**Notes:**
It's the last work of this year
In memory of the last time here, wish for a great step in 2018
May the force be with you
