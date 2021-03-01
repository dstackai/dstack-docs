---
description: Learn how to manage dependencies of your applications.
---

# Dependencies

A `dstack` application may contain its own packages and modules as well have dependencies to third-party libraries.

In order to push such an application, one must provide the information on these packages, modules, and libraries within the call of the `dstack.app()` function.

Here's an example:

```python
import dstack as ds
import dstack.controls as ctrl

from handlers import fake_handler

app = ds.app(outputs=[ctrl.Output(handler=fake_handler)], depends=["handlers", "utils"],
             requirements="requirements.txt")

url = ds.push("deps_app", app)
print(url)
```

As you see, we use `depends` and `requirements` arguments to specify what modules, packages, and libraries our application depends on. In this case, the application depends on the module \`handlers\`, the package `utils`, and on all libraries specified in the `requirements.txt`.

The `depends` argument may list either local modules and packages or PiPy packages. An alternative equivalent of the line above would be the following:

```python
ds.app(outputs=[ctrl.Output(handler=fake_handler)], depends=["numpy", "pandas", "faker==5.5.0", "handlers", "utils"])
```

{% hint style="warning" %}
Note, it's important that when you run `app.py` the root directory is `deps_app` where `handlers` and `utils` are located. If the directory is different, `dstack` may not find them.
{% endhint %}

When you run the application the first time, `dstack` makes sure all dependencies are installed.

{% hint style="info" %}
**Source Code:** [**https://github.com/dstackai/dstack-examples/tree/master/depends**](https://github.com/dstackai/dstack-examples/tree/master/depends)\*\*\*\*
{% endhint %}

{% page-ref page="../tutorials/simple-application-with-dependencies.md" %}



