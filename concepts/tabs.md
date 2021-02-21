---
description: Learn how to create multi-tab applications.
---

# Tabs

`dstack` allows applications to have multiple tabs. Here's an example of the application with two tabs: `"Scatter Chart"` and `"Bar Chart".`

In order to push an application with multiple tabs, one has to create a frame by using `dstack.frame()`, then call the function `add()` on the frame for every tab and pass there the corresponding `controls` and `outputs.` The title of the corresponding tab is passed with the attribute `params` , as dictionary with the title of the tab mapped to `ds.tab()`\(see the example above\). 

Finally, the function `push` must be called on the frame. Just like `dstack.push()`, it pushes the application and returns its URL. Here's the code:

```python
import dstack as ds
import dstack.controls as ctrl
import plotly.express as px


def scatter_handler(self: ctrl.Output):
    df = px.data.iris()
    self.data = px.scatter(df, x="sepal_width", y="sepal_length", color="species")


def bar_handler(self: ctrl.Output):
    df = px.data.tips()
    self.data = px.bar(df, x="sex", y="total_bill", color="smoker", barmode="group")


frame = ds.frame("minimal_app_tabs")

frame.add(ds.app(outputs=[ds.Output(handler=scatter_handler)]), params={"Scatter Chart": ds.tab()})
frame.add(ds.app(outputs=[ds.Output(handler=bar_handler)]), params={"Bar Chart": ds.tab()})

url = frame.push()
print(url)
```

If you push the application from the example above and then open its URL, you'll see the following:

![](../.gitbook/assets/ds_multi_tab.png)

Similar to applications with no tabs, every tab in a multi-tab application may have any number of controls or outputs.

