---
description: Learn how to improve performance of your applications by using cache.
---

# Caching

Since `dstack` applications often work with large datasets and ML models, unnecessary heavy computations may affect the performance of applications. In order to avoid it, `dstack` offers the concept of caching. 

Here's the most simple example of how it works:

```python
import dstack as ds

@ds.cache()
def get_data():
    return pd.read_csv("https://www.dropbox.com/s/cat8vm6lchlu5tp/data.csv?dl=1", index_col=0)
    
```

Now, if the function `get_data()` is called from an application, `dstack` will call it only once and then will re-use its value if it's called again.

The annotation `dstack.cache()` can be used on any function regardless of how many arguments the function has:

```python
import dstack as ds

@ds.cache()
def get_data(url):
    return pd.read_csv(url, index_col=0)
    
```

In the example above, for every unique value of `url`,  `dstack` will re-use the correct cached version.

`dstack` offers a way to customize the behavior of `dstack.cache()` by specifying a function that provides an object on which the hash of the function arguments is calculated. 

Imagine, you'd like to change the example above to invalidate the cache each day:

```python
import dstack as ds
from datetime import datetime

def my_hash_func(url):
    return (url, datetime.today().date())

@ds.cache(hash_func=my_hash_func)
def get_data(url):
    return pd.read_csv(url, index_col=0)
```

Here', we define the function `my_hash_func()`, which takes the same arguments as `get_data()`. This function returns a tuple with the `url` and today's date. Now, if the function get\_data is called, `dstack` invokes the function `my_hash_func()` on the given arguments and then calculates a hash on it. If there's a cached value associated with the calculated cache, `dstack` returns it and make an unnecessary call of `get_data()`. 

