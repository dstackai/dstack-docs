---
description: Learn how to manage dependencies of your applications.
---

# Dependencies

When building a `dstack` application, you can use your own packages and modules as well as third-party libraries.

The information on these packages, modules, and libraries must be specified within the call of the `dstack.app()` function. Here's an example:

```python
import dstack as ds

from handlers import fake_handler

# create an instance of an application an pass over 
# dependencies to local modules "handlers" and "utils" and third-party packages
app = ds.app(depends=["handlers", "utils"], requirements="requirements.txt")

# The line above is equal to the line below
# app = ds.app(depends=["numpy", "pandas", "faker==5.5.0", "handlers", "utils"])

# an output with a handler from one of the modules the application depends on
app.output(handler=fake_handler)

# deploy the application with the name "stocks" and print its URL
url = app.deploy("faker")
print(url)
```

As you see here, we use the `depends` and `requirements` arguments to specify what modules, packages, and libraries the application depends on. In this case, the application depends on the module `handlers`, the package `utils`, and on all libraries specified in the `requirements.txt`.

{% hint style="warning" %}
Be careful, when you run the script that deploys a dstack application with dependencies to local modules or packages, make sure that the root folder from where you run the script is exactly the folder that contains these packages and modules.
{% endhint %}

When you run the application the first time, `dstack` makes sure all dependencies are installed on the first run.

{% hint style="info" %}
**Source Code:** [**https://github.com/dstackai/dstack-examples/tree/master/depends**](https://github.com/dstackai/dstack-examples/tree/master/depends)\*\*\*\*
{% endhint %}

{% page-ref page="../tutorials/simple-application-with-dependencies.md" %}



