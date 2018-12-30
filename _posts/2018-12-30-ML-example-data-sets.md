---
layout: post
title: Scikit-learn working with example data sets
tags:
    - sklearn
    - machine learning
    - Python
author: Radosław Śmigielski
---
At the start of machine learning examples you will need some example data sets.
Scikit-learn provides few good examples, here is more what it provides and how
to use it.

## Scikit-learn example data sets ##
Scikit-learn comes with number of existing example data sets you can use
when starting with machine learning. This is how you can find them:
{% highlight bash %}
>>> from sklearn import datasets
>>> dir(datasets)
{% endhighlight %}
You can see three type of functions:
1. **make\_\<SOME_DATA_SET\>** - Generate samples of synthetic data sets.
1. **load\_\<SOME_DATA_SET\>** - Load local data sets which come with
   Scikit-learn. These are parts of python *sklearn* package.

   All available:
   - boston_house_prices.csv
   - breast_cancer.csv
   - diabetes_data.csv.gz
   - diabetes_target.csv.gz
   - digits.csv.gz
   - iris.csv
   - linnerud_exercise.csv
   - linnerud_physiological.csv
   - wine_data.csv
1. **fetch\_\<SOME_DATA_SET\>** - Load external data set,
   download them from Internet. These are too bit to include them
   into Python package and some of them include multiple binary files.

   All available:
   - fetch_20newsgroups
   - fetch_20newsgroups_vectorized
   - fetch_lfw_pairs
   - fetch_lfw_people
   - fetch_mldata
   - fetch_olivetti_faces
   - fetch_species_distributions
   - fetch_california_housing
   - fetch_covtype
   - fetch_rcv1
   - fetch_kddcup99
   - fetch_openml

   And this is how you can fetch some data:
   {% highlight bash %}
   >>> california_housing = datasets.fetch_california_housing()
   Downloading Cal. housing from https://ndownloader.figshare.com/files/5976036 to /home/radek/scikit_learn_data
   {% endhighlight %}

## Scikit-learn examples data format ##
   This is how you can load the data:
   {% highlight bash %}
   >>> bp = datasets.load_boston()
   >>> dir(bp)
   ['DESCR', 'data', 'feature_names', 'filename', 'target']
   >>> type(bp)
   <class 'sklearn.utils.Bunch'>
   {% endhighlight %}
   
   Where _sklearn.utils.Bunch_ is a dictionary-like object that exposes its
   keys as attributes.

