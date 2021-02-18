---
description: Learn which types of outputs are supported and how to generate them.
---

# Outputs

The output of the application is defined by one or several instances of `dstack.controls.Output` passed to `dstack.app()` as a part of the `outputs` attribute. 

The supported types of outputs include: 

* `pandas.core.generic.NDFrame` \(a Pandas dataframe\)
* `plotly.graph_objs._figure.Figure` \(a Plotly figure\)
* `matplotlib.figure.Figure` \(a Matplotlib figure\)
* `bokeh.plotting.Figure` \(a Bokeh figure\)
* `dstack.markdown.Markdown` \(a Markdown text, can be created via `dstack.md()`\)

Given the supported base types, it's possible to use `Pandas`, `Plotly`, `Bokeh`, `Matplotlib`, and `Seaborn` , and `Markdown` as outputs.

Let's take a look at the following example: 

```python
from datetime import datetime, timedelta

import dstack.controls as ctrl
import dstack as ds
import plotly.graph_objects as go
import pandas_datareader.data as web


def output_handler(self: ctrl.Output, symbols: ctrl.ComboBox):
    start = datetime.today() - timedelta(days=30)
    end = datetime.today()
    df = web.DataReader(symbols.value(), 'yahoo', start, end)
    fig = go.Figure(
        data=[go.Candlestick(x=df.index, open=df['Open'], high=df['High'], low=df['Low'], close=df['Close'])])
    self.data = fig


symbols = ctrl.ComboBox(items=["FB", "AMZN", "AAPL", "NFLX", "GOOG"])

app = ds.app(controls=[symbols],
             outputs=[ctrl.Output(data=ds.md("Here's a simple application with **Markdown** and a chart.")),
                      ctrl.Output(handler=output_handler, depends=[symbols])])

result = ds.push("minimal_app_with_md", app)
print(result.url)
```

For every output, you can pass either `data` or `handler` to define the value of the output. The difference between `data` and `handler` is the following:

* The `data` can be the end value of the output or a `typing.Callable` that returns that value output. If data is a `typing.Callable`, it shouldn't have any arguments.
* The handler is always a `typing.Callable` where the first argument is always self of the type `dstack.controls.Output`, and the rest of the arguments represent the controls the output depends on. In our case it's `symbols` of the type `dstack.controls.ComboBox`.

To see a more complex example of an application with multiple outputs, check out the following tutorial:

{% page-ref page="../tutorials/simple-application-with-multiple-outputs.md" %}



