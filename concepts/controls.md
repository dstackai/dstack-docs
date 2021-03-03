---
description: Learn which type of controls are supported and how to use them.
---

# Controls

## Overview

Controls are minimal building blocks of `dstack` applications. Controls allow the user of the application to change input parameters and see the corresponding outputs. An application may have any number of controls that may depend on each other. The user may define the Python code that initializes or updates the state of controls.

The supported controls include text inputs, single and multiple selection controls, sliders, check-boxes, file uploaders, markdown outputs, table outputs, chart output, and markdown outputs.

### Example

Here's a simple example:

```python
import dstack as ds
import plotly.express as px

app = ds.app()  # Create an instance of the application


# An utility function that loads the data
def get_data():
    return px.data.stocks()


# A drop-down control that shows stock symbols
stock = app.select(items=get_data().columns[1:].tolist())


# A handler that updates the plot based on the selected stock
def output_handler(self, stock):
    # A plotly line chart where the X axis is date and Y is the stock's price
    self.data = px.line(get_data(), x='date', y=stock.value())


# A plotly chart output
app.output(handler=output_handler, depends=[stock])

# Deploy the application with the name "stocks" and print its URL
url = app.deploy("stocks")
print(url)
```

If we run the code above and open the link, we'll see the following application:

**`TODO:`** `Add a screenshot`

### Control State

As you saw above, you can initialize a control by setting either the initial state of the control \(e.g. by setting `items` to `select`\), or a `handler` \(as we did above for `output`\). 

Here's the example of setting the state directly \(note, you can pass values or even a function that returns them\):

```python
# A drop-down control that shows stock symbols
stock = app.select(items=get_data().columns[1:].tolist())
```

Here's the example of using a handler:

```python
# A handler that updates the plot based on the selected stock
def output_handler(self, stock):
    # A plotly line chart where the X axis is date and Y is the stock's price
    self.data = px.line(get_data(), x="date", y=stock.value())


# A plotly chart output
app.output(handler=output_handler, depends=[stock])
```

A handler is a function in which the first argument is always self of the same type as the control of the handler. In case the control depends on other controls, these controls are passed to the handler too.

As soon as the stock control gets updates, `dstack` invokes the handler to update the state of the output. Inside the handler, it's possible to update any field of the control.

Now, let's look at a more complicated example, where the items of the drop-down control are populated dynamically, and which has another drop-down that depends on the first drop-down control:

```python
import dstack as ds
import pandas as pd

app = ds.app()  # Create an instance of the application


# An utility function that loads the data
def get_data():
    return pd.read_csv("https://www.dropbox.com/s/cat8vm6lchlu5tp/data.csv?dl=1", index_col=0)


# An utility function that returns regions
def get_regions():
    df = get_data()
    return df["Region"].unique().tolist()


# A drop-down control that shows regions
regions = app.select(items=get_regions, label="Region")


# A handler that updates the drop-down with counties based on the selected region
def countries_handler(self, regions):
    region = regions.value()  # the selected region
    df = get_data()
    self.items = df[df["Region"] == region]["Country"].unique().tolist()


# A drop-down control that shows countries
countries = app.select(handler=countries_handler, label="Country", depends=[regions])


# A handler that updates the table output based on the selected country
def output_handler(self, countries):
    country = countries.value()  # the selected country 
    df = get_data()
    self.data = df[df["Country"] == country]  # we assign a pandas dataframe here to self.data


# An output that shows companies based on the selected country
app.output(handler=output_handler, depends=[countries])

# Deploy the application with the name "dependant_control" and print its URL
url = app.deploy("dependant_control")
print(url)
```

If you run this code and open the application, you'll see the following:

**`TODO:`** `Add screenshot`

### Sidebar

By default, all controls are placed in the main area. Sometimes, it may be useful to place certain control in a sidebar. To do that, you have to use the `dstack.Application.sidebar()` function. Here's a very simple example:

```python
import dstack as ds
import plotly.express as px

app = ds.app()  # create an instance of the application

siebar = app.sidebar() # create a sidebar

# an utility function that loads the data
def get_data():
    return px.data.stocks()


# a drop-down control inside the sidebar
stock = siebar.select(items=get_data().columns[1:].tolist())


# a handler that updates the plot based on the selected stock
def output_handler(self, stock):
    # a plotly line chart where the X axis is date and Y is the stock's price
    self.data = px.line(get_data(), x='date', y=stock.value())


# a plotly chart output in the main area
app.output(handler=output_handler, depends=[stock])

# deploy the application with the name "stocks_sidebar" and print its URL
url = app.deploy("stocks_sidebar")
print(url)
```

Now, if you open the application, you'll see the following:

**`TODO:`** `Add screenshot`

### Require Apply

By default, the application triggers the application update \(including updating outputs\) every time the user changes anything. If you'd like any control to update only if the user clicks `Apply`, you can set `requires_apply=True` when you create a control:

```python
# An output that shows companies based on the selected country
app.output(handler=output_handler, depends=[countries], require_apply=True)
```

Now, if you open the application, you'll see the following:

**`TODO:`** `Add a screenshot`

**`TODO:`** `Add a link to gallery`

**`TODO:`** `Add a link to GitHub repo`

### Layout

Controls may be visually arranged within the application using the `layout` system. Currently, `dstack` supports only the `"grid"` layout. This means an application is divided into columns and rows, and controls may take up any number of these columns and rows.

Here's how it works:

```python
import dstack as ds

# create an instance of the application that has three columns
app = ds.app(columns = 3) 

# an input that takes one column and one row
input_1 = app.input(label="Input 1", colspan=1)
# an input that takes one column and one row
input_2 = app.input(label="Input 2", colspan=1)
# an input that takes one column and two rows
input_3 = app.input(label="Input 3", colspan=1, rowspan=2)

url = app.deploy("layout_1")
print(url)
```

If we open the application, we'll see the following:

**`TODO:`** `Add screenshot`

If you don't specify `columns` within `dstack.app()`, it will be set to `12`.

{% hint style="info" %}
The sidebar just like the main area also uses the `"grid"` layout. However, by default, it has `2` columns instead of `12`. 
{% endhint %}

## Control Reference

Below, you'll find the entire list of supported controls, with a complete list of their attributes, and examples.

### Input

The `Input` control can be used to enter text:

```python
import dstack as ds

app = ds.app()  # Create an instance of the application


# A handler that updates the markdown output based on the input text
def markdown_handler(self, name):
    if len(name.text) > 0:
        self.text = "Hi, **" + name.text + "**!"
    else:
        self.text = "No name"


# An input control
name = app.input(label="What's your name?")

# A markdown output that greets the users using the given name
app.markdown(handler=markdown_handler, depends=[name])

# Deploy the application with the name "controls/input" and print its URL
url = app.deploy("controls/input")
print(url)
```

**`TODO:`** `Add screenshot`

Here's the list of arguments of the `dstack.Application.input()` function:

<table>
  <thead>
    <tr>
      <th style="text-align:left">Parameter</th>
      <th style="text-align:left">Type</th>
      <th style="text-align:left">Description</th>
      <th style="text-align:left">Required</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><code>text</code>
      </td>
      <td style="text-align:left">
        <p>Can be one of the following:</p>
        <ul>
          <li><code>str</code>
          </li>
          <li><code>Callable[[], str]</code>
          </li>
        </ul>
      </td>
      <td style="text-align:left">The text value of the control.</td>
      <td style="text-align:left">Not required if <code>handler</code> is used.</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>handler</code>
      </td>
      <td style="text-align:left"><code>Callable[..., None]</code>
      </td>
      <td style="text-align:left">The function that initializes or updates the state of the control.</td>
      <td
      style="text-align:left">Required if <code>text</code> is not set.</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>label</code>
      </td>
      <td style="text-align:left"><code>str</code>
      </td>
      <td style="text-align:left">The caption of the control.</td>
      <td style="text-align:left">No</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>depends</code>
      </td>
      <td style="text-align:left">
        <p>Can be one of the following:</p>
        <ul>
          <li><code>List[Control]</code>
          </li>
          <li><code>Control</code>
          </li>
        </ul>
      </td>
      <td style="text-align:left">The other controls this control depends on.</td>
      <td style="text-align:left">No</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>colspan</code>
      </td>
      <td style="text-align:left"><code>int</code>
      </td>
      <td style="text-align:left">The number of columns the control is taking up. By default, it&apos;s <code>2</code>.</td>
      <td
      style="text-align:left">No</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>rowspan</code>
      </td>
      <td style="text-align:left"><code>int</code>
      </td>
      <td style="text-align:left">The number of rows the control is taking up. By default, it&apos;s <code>1</code>.</td>
      <td
      style="text-align:left">No</td>
    </tr>
  </tbody>
</table>

**`TODO:`** `Add a screenshot`

### Select

The `Select` control can be used to select one or multiple items:

```python
import dstack as ds
import plotly.express as px

app = ds.app()  # create an instance of the application


# an utility function that loads the data
def get_data():
    return px.data.stocks()


# a drop-down control that shows stock symbols
stock = app.select(items=get_data().columns[1:].tolist())


# a handler that updates the plot based on the selected stock
def output_handler(self, stock):
    symbol = stock.value()  # the selected stock
    # a plotly line chart where the X axis is date and Y is the stock's price
    self.data = px.line(get_data(), x='date', y=symbol)


# a plotly chart output
app.output(handler=output_handler, depends=[stock])

# deploy the application with the name "stocks" and print its URL
url = app.deploy("stocks")
print(url)
```

**`TODO:`** `Add a screenshot`

Here's the list of arguments of the `dstack.Application.select()` function:

<table>
  <thead>
    <tr>
      <th style="text-align:left">Parameter</th>
      <th style="text-align:left">Type</th>
      <th style="text-align:left">Description</th>
      <th style="text-align:left">Required</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><code>items</code>
      </td>
      <td style="text-align:left">
        <p></p>
        <p>Can be one of the following:</p>
        <ul>
          <li><code>List[Any]</code>
          </li>
          <li><code>Callable</code>
          </li>
        </ul>
        <p></p>
      </td>
      <td style="text-align:left">
        <p></p>
        <p>Can be one of the following:</p>
        <ul>
          <li>A list of items.</li>
          <li>A function that returns a list of items. <em>See example B.</em>
          </li>
        </ul>
      </td>
      <td style="text-align:left">Not required if <code>handler</code> is used.</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>handler</code>
      </td>
      <td style="text-align:left"><code>Callable[..., None]</code>
      </td>
      <td style="text-align:left">The function that initializes or updates the state of the control.</td>
      <td
      style="text-align:left">Required if <code>items</code> is not set.</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>selected</code>
      </td>
      <td style="text-align:left">
        <p>Can be one of the following:</p>
        <ul>
          <li><code>int</code>
          </li>
          <li><code>List[int]</code>
          </li>
        </ul>
      </td>
      <td style="text-align:left">
        <p>Can be one of the following:</p>
        <ul>
          <li>An index of the currently selected item. Only if <code>multiple</code> set
            to <code>False</code>.</li>
          <li>A list of indexes of the currently selected items. Only if <code>multiple</code> set
            to <code>True</code>.</li>
        </ul>
      </td>
      <td style="text-align:left">No</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>multiple</code>
      </td>
      <td style="text-align:left"><code>bool</code>
      </td>
      <td style="text-align:left"><code>True</code> if multiple selection is allowed. <code>False</code> by
        default.</td>
      <td style="text-align:left">No</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>label</code>
      </td>
      <td style="text-align:left"><code>str</code>
      </td>
      <td style="text-align:left">The caption of the control.</td>
      <td style="text-align:left">No</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>depends</code>
      </td>
      <td style="text-align:left">
        <p>Can be one of the following:</p>
        <ul>
          <li><code>List[Control]</code>
          </li>
          <li><code>Control</code>
          </li>
        </ul>
      </td>
      <td style="text-align:left">The other controls this control depends on.</td>
      <td style="text-align:left">No</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>colspan</code>
      </td>
      <td style="text-align:left"><code>int</code>
      </td>
      <td style="text-align:left">The number of columns the control is taking up. By default, it&apos;s <code>2</code>.</td>
      <td
      style="text-align:left">No</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>rowspan</code>
      </td>
      <td style="text-align:left"><code>int</code>
      </td>
      <td style="text-align:left">The number of rows the control is taking up. By default, it&apos;s <code>1</code>.</td>
      <td
      style="text-align:left">No</td>
    </tr>
  </tbody>
</table>

### Checkbox

The `Checkbox` control can used to select or unselect a certain `boolean` property. Here's an example:

```python
import dstack as ds

app = ds.app()  # Create an instance of the application


# A handler that updates the label of the checkbox based on wether it's selected or not
def checkbox_handler(self):
    if self.selected:
        self.label = "Selected"
    else:
        self.label = "Not selected"


# A checkbox control
name = app.checkbox(handler=checkbox_handler)

# Deploy the application with the name "controls/checkbox" and print its URL
url = app.deploy("controls/checkbox")
print(url)
```

**`TODO:`** `Add a screenshot`

Here's the list of arguments of the `dstack.Application.checkbox()` function:

<table>
  <thead>
    <tr>
      <th style="text-align:left">Parameter</th>
      <th style="text-align:left">Type</th>
      <th style="text-align:left">Description</th>
      <th style="text-align:left">Required</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><code>selected</code>
      </td>
      <td style="text-align:left">
        <p>Can be one of the following:</p>
        <ul>
          <li><code>bool</code>
          </li>
          <li><code>Callable</code>
          </li>
        </ul>
      </td>
      <td style="text-align:left">The initial value of the control.</td>
      <td style="text-align:left">Not required if <code>handler</code> is used.</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>handler</code>
      </td>
      <td style="text-align:left"><code>Callable</code>
      </td>
      <td style="text-align:left">The function that initializes or updates the state of the control.</td>
      <td
      style="text-align:left">Required if <code>selected</code> is not set.</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>label</code>
      </td>
      <td style="text-align:left"><code>str</code>
      </td>
      <td style="text-align:left">The caption of the control.</td>
      <td style="text-align:left">No</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>depends</code>
      </td>
      <td style="text-align:left">
        <p>Can be one of the following:</p>
        <ul>
          <li><code>List[Control]</code>
          </li>
          <li><code>Control</code>
          </li>
        </ul>
      </td>
      <td style="text-align:left">The other controls this control depends on.</td>
      <td style="text-align:left">No</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>colspan</code>
      </td>
      <td style="text-align:left"><code>int</code>
      </td>
      <td style="text-align:left">The number of columns the control is taking up. By default, if the checkbox
        has a <code>label</code>, it&apos;s <code>2</code>. Otherwise, it&apos;s <code>1</code>.</td>
      <td
      style="text-align:left">No</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>rowspan</code>
      </td>
      <td style="text-align:left"><code>int</code>
      </td>
      <td style="text-align:left">The number of rows the control is taking up. By default, it&apos;s <code>1</code>.</td>
      <td
      style="text-align:left">No</td>
    </tr>
  </tbody>
</table>

### Slider

The `Slider` control can be used to select a number out of a given range. It supports both integer and double numbers. Here's an example:

```python
import dstack as ds
import plotly.express as px

app = ds.app()  # Create an instance of the application


# An utility function that loads the data
def get_data():
    return px.data.gapminder()


# A handler that updates the plot output based on the selected year
def output_handler(self, year):
    value = year.value()  # the selected year
    self.data = px.scatter(get_data().query("year==" + str(value)), x="gdpPercap", y="lifeExp",
                           size="pop", color="country", hover_name="country", log_x=True, size_max=60)


# A slider control that prompts to select a year
slider = app.slider(values=get_data()["year"].unique().tolist())

# An output control that shows the chart
app.output(handler=output_handler, depends=[slider])

# Deploy the application with the name "controls/slider" and print its URL
url = app.deploy("controls/slider")
print(url)
```

**`TODO:`** `Add a screenshot`

Here's the list of arguments of the `dstack.Application.slider()` function:

<table>
  <thead>
    <tr>
      <th style="text-align:left">Parameter</th>
      <th style="text-align:left">Type</th>
      <th style="text-align:left">Description</th>
      <th style="text-align:left">Required</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><code>values</code>
      </td>
      <td style="text-align:left">
        <p></p>
        <p>Can be one of the following:</p>
        <ul>
          <li><code>Iterable[float]</code>
          </li>
          <li><code>Callable</code>
          </li>
        </ul>
        <p></p>
      </td>
      <td style="text-align:left">
        <p></p>
        <p>Can be one of the following:</p>
        <ul>
          <li>A list of possible values. <em>See example A.</em>
          </li>
          <li>A function that returns a list of possible values. <em>See example B.</em>
          </li>
        </ul>
      </td>
      <td style="text-align:left">Not required if <code>handler</code> is used.</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>handler</code>
      </td>
      <td style="text-align:left"><code>Callable[..., None]</code>
      </td>
      <td style="text-align:left">The function that initializes or updates the state of the control.</td>
      <td
      style="text-align:left">Required if <code>items</code> is not set.</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>selected</code>
      </td>
      <td style="text-align:left">
        <p>Can be one of the following:</p>
        <ul>
          <li><code>int</code>
          </li>
          <li><code>List[int]</code>
          </li>
        </ul>
      </td>
      <td style="text-align:left">
        <p>Can be one of the following:</p>
        <ul>
          <li>An index of the currently selected item. Only if <code>multiple</code> set
            to <code>False</code>.</li>
          <li>A list of indexes of the currently selected items. Only if <code>multiple</code> set
            to <code>True</code>.</li>
        </ul>
      </td>
      <td style="text-align:left">No</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>label</code>
      </td>
      <td style="text-align:left"><code>str</code>
      </td>
      <td style="text-align:left">The caption of the control.</td>
      <td style="text-align:left">No</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>depends</code>
      </td>
      <td style="text-align:left">
        <p>Can be one of the following:</p>
        <ul>
          <li><code>List[Control]</code>
          </li>
          <li><code>Control</code>
          </li>
        </ul>
      </td>
      <td style="text-align:left">The other controls this control depends on.</td>
      <td style="text-align:left">No</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>colspan</code>
      </td>
      <td style="text-align:left"><code>int</code>
      </td>
      <td style="text-align:left">The number of columns the control is taking up. By default, it&apos;s <code>2</code>.</td>
      <td
      style="text-align:left">No</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>rowspan</code>
      </td>
      <td style="text-align:left"><code>int</code>
      </td>
      <td style="text-align:left">The number of rows the control is taking up. By default, it&apos;s <code>1</code>.</td>
      <td
      style="text-align:left">No</td>
    </tr>
  </tbody>
</table>

### Uploader

The `Uploader` control can be used to upload single or multiple files and access their content. Here's an example:

```python
import dstack as ds
import pandas as pd

app = ds.app()  # Create an instance of the application


# A handler that loads a dataframe from the content of the uploaded CSV file and passes it to the output
def app_handler(self, uploader):
    if len(uploader.uploads) > 0:
        with uploader.uploads[0].open() as f:
            self.label = uploader.uploads[0].file_name
            self.data = pd.read_csv(f).head(100)
    else:
        self.label = "No file selected"
        self.data = None


# A file uploader control
uploader = app.uploader(label="Select a CSV file")

# An output control that shows the content of the uploaded file
app.output(handler=app_handler, depends=[uploader])

# Deploy the application with the name "controls/select" and print its URL
url = app.deploy("controls/file_uploader")
print(url)
```

**`TODO:`** `Add a screenshot`

Here's the list of arguments of the `dstack.Application.uploader()` function:

<table>
  <thead>
    <tr>
      <th style="text-align:left">Parameter</th>
      <th style="text-align:left">Type</th>
      <th style="text-align:left">Description</th>
      <th style="text-align:left">Required</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><code>uploads</code>
      </td>
      <td style="text-align:left">
        <p>Can be one of the following:</p>
        <ul>
          <li><code>List[Upload]</code>
          </li>
          <li><code>Callable[[], List[Upload]]</code>
          </li>
        </ul>
      </td>
      <td style="text-align:left">The list of uploaded files.</td>
      <td style="text-align:left">Not required if <code>handler</code> is used.</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>handler</code>
      </td>
      <td style="text-align:left"><code>Callable[..., None]</code>
      </td>
      <td style="text-align:left">The function that initializes or updates the state of the control.</td>
      <td
      style="text-align:left">Required if <code>uploads</code> is not set.</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>multiple</code>
      </td>
      <td style="text-align:left"><code>bool</code>
      </td>
      <td style="text-align:left"><code>True</code> if multiple files are allowed. <code>False</code> by default.</td>
      <td
      style="text-align:left">No</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>label</code>
      </td>
      <td style="text-align:left"><code>str</code>
      </td>
      <td style="text-align:left">The caption of the control.</td>
      <td style="text-align:left">No</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>depends</code>
      </td>
      <td style="text-align:left">
        <p>Can be one of the following:</p>
        <ul>
          <li><code>List[Control]</code>
          </li>
          <li><code>Control</code>
          </li>
        </ul>
      </td>
      <td style="text-align:left">The other controls this control depends on.</td>
      <td style="text-align:left">No</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>colspan</code>
      </td>
      <td style="text-align:left"><code>int</code>
      </td>
      <td style="text-align:left">The number of columns the control is taking up. By default, it&apos;s <code>2</code>.</td>
      <td
      style="text-align:left">No</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>rowspan</code>
      </td>
      <td style="text-align:left"><code>int</code>
      </td>
      <td style="text-align:left">The number of rows the control is taking up. By default, it&apos;s <code>1</code>.</td>
      <td
      style="text-align:left">No</td>
    </tr>
  </tbody>
</table>

### Markdown

The `Markdown` control can be used to display a markdown text. Here's the same example that we used earlier:

```python
import dstack as ds

app = ds.app()  # create an instance of the application


# a handler that updates the markdown output based on the input text
def markdown_handler(self, name):
    if len(name.text) > 0:
        self.text = "Hi, **" + name.text + "**!"
    else:
        self.text = "No name"


# an input control
name = app.input(label="What's your name?")

# a markdown output that greets the users using the given name
app.markdown(handler=markdown_handler, depends=[name])

# deploy the application with the name "controls/input" and print its URL
url = app.deploy("controls/input")
print(url)
```

Above, you see a `Markdown` control that displays the text based on the text specified in another control. Below is a more simple example, where the `Markdown` control just displays a static text:

```python
import dstack as ds

app = ds.app()  # Create an instance of the application

# A markdown output
app.markdown(text="Hello, **World!**")

# Deploy the application with the name "markdown" and print its URL
url = app.deploy("markdown")
print(url)
```

**`TODO:`** `Add a screenshot`

Here's the list of arguments of the `dstack.Application.markdown()` function:

<table>
  <thead>
    <tr>
      <th style="text-align:left">Parameter</th>
      <th style="text-align:left">Type</th>
      <th style="text-align:left">Description</th>
      <th style="text-align:left">Required</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><code>text</code>
      </td>
      <td style="text-align:left">
        <p>Can be one of the following:</p>
        <ul>
          <li><code>str</code>
          </li>
          <li><code>Callable[[], str]</code>
          </li>
        </ul>
      </td>
      <td style="text-align:left">The text value of the control.</td>
      <td style="text-align:left">Not required if <code>handler</code> is used.</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>handler</code>
      </td>
      <td style="text-align:left"><code>Callable[..., None]</code>
      </td>
      <td style="text-align:left">The function that initializes or updates the state of the control.</td>
      <td
      style="text-align:left">Required if <code>text</code> is not set.</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>label</code>
      </td>
      <td style="text-align:left"><code>str</code>
      </td>
      <td style="text-align:left">The caption of the control.</td>
      <td style="text-align:left">No</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>depends</code>
      </td>
      <td style="text-align:left">
        <p>Can be one of the following:</p>
        <ul>
          <li><code>List[Control]</code>
          </li>
          <li><code>Control</code>
          </li>
        </ul>
      </td>
      <td style="text-align:left">The other controls this control depends on.</td>
      <td style="text-align:left">No</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>requires_apply</code>
      </td>
      <td style="text-align:left"><code>bool</code>
      </td>
      <td style="text-align:left"><code>True</code> if the field requires an <code>Apply</code> button to be
        clicked for the control to update. <code>False</code> by default.</td>
      <td
      style="text-align:left">No</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>colspan</code>
      </td>
      <td style="text-align:left"><code>int</code>
      </td>
      <td style="text-align:left">The number of columns the control is taking up. By default, it&apos;s <code>12</code>.</td>
      <td
      style="text-align:left">No</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>rowspan</code>
      </td>
      <td style="text-align:left"><code>int</code>
      </td>
      <td style="text-align:left">The number of rows the control is taking up. By default, it&apos;s <code>6</code>.</td>
      <td
      style="text-align:left">No</td>
    </tr>
  </tbody>
</table>

### Output

The `Output` control can be used to display table data or charts. Outputs support `Pandas` dataframes, and `Plotly`, `Bokeh`, `Matplotlib`, and `Seaborn` charts.

#### Pandas

**`TODO:`** `Add Pandas example`

#### Plotly

**`TODO:`** `Add Plotly example`

#### Bokeh

**`TODO:`** `Add Bokeh example`

#### Matplotlib

**`TODO:`** `Add Pandas example`

#### Seaborn

**`TODO:`** `Add Seaborn example`

Here's the list of arguments of the `dstack.Application.output()` function:

<table>
  <thead>
    <tr>
      <th style="text-align:left">Parameter</th>
      <th style="text-align:left">Type</th>
      <th style="text-align:left">Description</th>
      <th style="text-align:left">Required</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><code>data</code>
      </td>
      <td style="text-align:left">
        <p>Can be one of the following:</p>
        <ul>
          <li><code>pandas.core.frame.DataFrame</code>
          </li>
          <li><code>plotly.basedatatypes.BaseFigure</code>
          </li>
          <li><code>matplotlib.figure.Figure</code>
          </li>
          <li><code>bokeh.plotting.Figure</code>
          </li>
          <li><code>Callable</code>
          </li>
        </ul>
      </td>
      <td style="text-align:left">The data to display in the output.</td>
      <td style="text-align:left">Not required if <code>handler</code> is used.</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>handler</code>
      </td>
      <td style="text-align:left"><code>Callable[..., None]</code>
      </td>
      <td style="text-align:left">The function that initializes or updates the state of the control.</td>
      <td
      style="text-align:left">Required if <code>text</code> is not set.</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>label</code>
      </td>
      <td style="text-align:left"><code>str</code>
      </td>
      <td style="text-align:left">The caption of the control.</td>
      <td style="text-align:left">No</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>depends</code>
      </td>
      <td style="text-align:left">
        <p>Can be one of the following:</p>
        <ul>
          <li><code>List[Control]</code>
          </li>
          <li><code>Control</code>
          </li>
        </ul>
      </td>
      <td style="text-align:left">The other controls this control depends on.</td>
      <td style="text-align:left">No</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>requires_apply</code>
      </td>
      <td style="text-align:left"><code>bool</code>
      </td>
      <td style="text-align:left"><code>True</code> if the field requires an <code>Apply</code> button to be
        clicked for the control to update. <code>False</code> by default.</td>
      <td
      style="text-align:left">No</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>colspan</code>
      </td>
      <td style="text-align:left"><code>int</code>
      </td>
      <td style="text-align:left">The number of columns the control is taking up. By default, it&apos;s <code>6</code>.</td>
      <td
      style="text-align:left">No</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>rowspan</code>
      </td>
      <td style="text-align:left"><code>int</code>
      </td>
      <td style="text-align:left">The number of rows the control is taking up. By default, it&apos;s <code>6</code>.</td>
      <td
      style="text-align:left">No</td>
    </tr>
  </tbody>
</table>

