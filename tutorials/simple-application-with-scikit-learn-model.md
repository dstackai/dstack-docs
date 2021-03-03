---
description: >-
  Here's an example of pushing ML model to dstack's ML Registry and using this
  model in an application.
---

# Simple Application with a Scikit-learn ML Model

`dstack` decouples the development of applications from the development of ML models by offering an ML registry. This way, one can develop ML models, push them to the registry, and then later pull these models from applications. 

In this tutorial, we'll use `sklearn` to build a Logistic Regression model. Since we want the model to be re-used on live data, we need to bundle together not only a model but also all pre-requisite steps that are required to transform the live data before training or predicting to bundle these steps together, we'll use `sklearn`'s [pipeline](https://scikit-learn.org/stable/modules/compose.html) feature.

Each step of the pipeline that we'll build will be described as a separate class that inherits `sklearn.base.BaseEstimator` and `sklearn.base.TransformerMixin`.

```python
import pandas as pd
from sklearn.base import BaseEstimator, TransformerMixin
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import train_test_split
from sklearn.pipeline import make_pipeline
import dstack as ds

# The first step transforms the data into the form suitable for model.
class PrepareData(BaseEstimator, TransformerMixin):
    def __init__(self):
        pass

    def transform(self, X, **transform_params):
        df = X.copy()

        # Add new features: "Years" (the number of years with a positive number of purchased licences)
        def years(row):
            l = [row["y2019"], row["y2018"], row["y2017"], row["y2016"], row["y2015"]]
            return len([x for x in l if x != 0])

        df["Years"] = df.apply(years, axis=1)

        # Drop features that aren't needed
        df = df.drop(["Company", "Region", "Manager", "RenewalMonth", "RenewalDate"], axis=1)

        # Normalize the values of the columns that need it
        for col in ["y2015", "y2016", "y2017", "y2018", "y2019"]:
            df[col] = df[col] / df[col].max()

        # Encode categorical columns into the columns with 0 and 1 -s (required by Logistic Regression)
        for c in X["Country"].unique():
            df[c] = df["Country"].apply(lambda x: 1 if x == c else 0)
        for s in X["Sector"].unique():
            if s:
                df[s] = df["Sector"].apply(lambda x: 1 if x == s else 0)
        df = df.drop(["Country", "Sector"], axis=1)

        return df

    def fit(self, X, y=None, **fit_params):
        return self


# The second step makes sure the data has exactly the same format as the data used to train the model.
# This step is necessary because the model that will be trained as the part of this pipeline, will
# be re-used later to make predictions based on other data.
class ReindexColumns(BaseEstimator, TransformerMixin):
    def __init__(self, columns):
        self.columns = columns  # the columns of the original data that was used to train the model

    def transform(self, X, **transform_params):
        # Drop the columns that were not present in the original data
        # Add missing columns that were present in the original data with 0 as value
        return X.reindex(columns=self.columns, fill_value=0)

    def fit(self, X, y=None, **fit_params):
        return self


# Read the data. Drop the rows with incomplete data (including the rows without historical churn data).
df = pd.read_csv("https://www.dropbox.com/s/cat8vm6lchlu5tp/data.csv?dl=1", index_col=0).dropna()
X = df.drop(["Churn"], axis=1)  # features
y = df["Churn"]  # target variable

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=99)

# Save the columns of the original processed data to be used in the pipeline.
X_train_columns = PrepareData().transform(X_train)

# Define the pipeline as a chain of steps
pipeline = make_pipeline(
    PrepareData(),  # 1. Transform the data into the form suitable for the model
    ReindexColumns(X_train_columns.columns),
    # 2 Make sure the transformed data has the same format as the original data that was used to train the model
    LogisticRegression()  # Pass the transformed and re-indexed data to the logistic regression
)
# Train the pipeline
pipeline.fit(X_train, y_train)

# deploy the pipeline with the name "tutorials/sklearn_model" and print its URL
url = ds.push("tutorials/sklearn_model", pipeline)
print(url)

```

If we run this code, it will train the model and push it to `dstack`. In the output, you'll see its URL which you can use to view the model via the interface. If you click it, you'll see the following:

**`TODO:`** `Add screenshot`

**`TODO:`** `Add link to gallery`

Now, this model is stored with `dstack`'s registry and can be pulled from any application. Let's look at an example of an application that uses this model:

```python
import dstack as ds
import numpy as np
import pandas as pd


# Retrieve the latest version of the model
def get_model():
    return ds.pull("tutorials/sklearn_model")


# An utility function that loads the data
@ds.cache()  # caching the result
def get_data():
    df = pd.read_csv("https://www.dropbox.com/s/cat8vm6lchlu5tp/data.csv?dl=1", index_col=0)
    df = df[df["Churn"].isnull()].drop(["Churn"], axis=1)
    return df


# An utility function that loads data, invokes the model to predict churn, and drops unnecessary columns
@ds.cache()
def get_predicted_data():
    df = get_data().copy()

    predicted_churn = get_model().predict(df)  # Predict churn

    # Replace 1.0 with "Yes" and 0.0 with "No"
    df["Predicted Churn"] = np.array(list(map(lambda x: "Yes" if x == 1.0 else "No", predicted_churn)))

    # Replace the numbers of months with the names of the months
    df["RenewalMonth"] = df["RenewalMonth"].apply(
        lambda x: ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'][x - 1])

    # Drop unnecessary columns
    return df.drop(["y2015", "y2016", "y2017", "y2018", "y2019"], axis=1)


# Create an application
app = ds.app()

# A drop-down control with the list of regions
regions_select = app.select(get_data()["Region"].unique().tolist(), label="Region")
# A drop drown controls with the list of months
months_select = app.select(['Oct', 'Nov', 'Dec'], label="Month")


# A handler that updates the state of the output with the predicted data
def app_handler(self, regions_select, months_select):
    df = get_predicted_data().copy()

    df = df[df["Predicted Churn"] == "Yes"]
    df = df[(df["Region"] == regions_select.value())]
    df = df[(df["RenewalMonth"] == months_select.value())]
    self.data = df


# A table output that shows the predicted data based on the selected region and month
app.output(handler=app_handler, depends=[regions_select, months_select])

# deploy the application with the name "sklearn" and print its URL
url = app.deploy("sklearn")
print(url)
```

Now, if we run this code, and open the URL from the output, we'll see the following:

**`TODO:`** `Add a screenshot`

**`TODO:`** `Add a link to gallery`

Now, if you push another version of the model using the same name, the application will switch to the new version of the model.

{% hint style="info" %}
**Source Code:** [**https://github.com/dstackai/dstack-examples/tree/master/sklearn**](https://github.com/dstackai/dstack-examples/tree/master/sklearn)\*\*\*\*
{% endhint %}

`dstack` supports `Tensorflow`, `PyTorch`, or `Scikit-Learn` models.

