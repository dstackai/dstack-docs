---
description: Learn which type of controls are supported and how to use them.
---

# Controls

Controls are building blocks of applications. They allow the user of the application to change the input parameters and display the corresponding outputs. The supported controls include text inputs, single and multiple selection controls, sliders, check-boxes, file uploaders, markdown outputs, table outputs, chart output, and markdown outputs.  Any control may depend on other controls. An application may have any number of controls, and these controls are arranged visually using the `Grid` layout. 

## Example

Here's a simple example:

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
    # a plotly line chart where the X axis is date and Y is the stock's price
    self.data = px.line(get_data(), x='date', y=stock.value())


# a plotly chart output
app.output(handler=output_handler, depends=[stock])

# deploy the application with the name "stocks" and print its URL
url = app.deploy("stocks")
print(url)
```

If we run the code above and open the link, we'll see the following application:

![](../.gitbook/assets/dstack_stocks.png)

In the example above we have a drop-down control, where we pass the list of items, and we have a chart output that is dependant on the drop-down control. To define a control that depends on other controls, you have to pass a handler, and the list of controls it's supposed to depend on:

```python
# a handler that updates the plot based on the selected stock
def output_handler(self, stock):
    # a plotly line chart where the X axis is date and Y is the stock's price
    self.data = px.line(get_data(), x='date', y=stock.value())


# a plotly chart output
app.output(handler=output_handler, depends=[stock])
```

Now, let's look at a more complicated example, where the items of the drop-down control are populated dynamically, and which has another drop-down that depends on the first drop-down control:

```python
import dstack as ds
import pandas as pd

app = ds.app()  # create an instance of the application


# an utility function that loads the data
def get_data():
    return pd.read_csv("https://www.dropbox.com/s/cat8vm6lchlu5tp/data.csv?dl=1", index_col=0)


# an utility function that returns regions
def get_regions():
    df = get_data()
    return df["Region"].unique().tolist()


# a drop-down control that shows regions
regions = app.select(items=get_regions, label="Region")


# a handler that updates the drop-down with counties based on the selected region
def countries_handler(self, regions):
    region = regions.value()  # the selected region
    df = get_data()
    self.items = df[df["Region"] == region]["Country"].unique().tolist()


# a drop-down control that shows countries
countries = app.select(handler=countries_handler, label="Country", depends=[regions])


# a handler that updates the table output based on the selected country
def output_handler(self, countries):
    country = countries.value()  # the selected country 
    df = get_data()
    self.data = df[df["Country"] == country]  # we assign a pandas dataframe here to self.data


# an output that shows companies based on the selected country
app.output(handler=output_handler, depends=[countries])

# deploy the application with the name "dependant_control" and print its URL
url = app.deploy("dependant_control")
print(url)
```

If you run this code and open the application using the URL from the output, you'll see the following application:

![](../.gitbook/assets/ds_dependant_controls_app_open_popup.png)

If you'd like the application to show the output only if the user clicks `Apply`, you can invoke the following code before deploying the application:

```python
app = ds.app(require_apply=True)
```

![](../.gitbook/assets/ds_dependant_controls_app_apply.png)

## Control Layout

**`TODO:`** `Add layout description`

## Control API Reference

### Input

The `Input` control can be used to enter text:

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
      <td style="text-align:left"><code>require_apply</code>
      </td>
      <td style="text-align:left"><code>bool</code>
      </td>
      <td style="text-align:left"><code>True</code> if the field requires an <code>Apply</code> button to be
        clicked for the application to update the output. <code>True</code> by default.</td>
      <td
      style="text-align:left">No</td>
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

app = ds.app()  # create an instance of the application


# a handler that updates the label of the checkbox based on wether it's selected or not
def checkbox_handler(self):
    if self.selected:
        self.label = "Selected"
    else:
        self.label = "Not selected"


# a checkbox control
name = app.checkbox(handler=checkbox_handler)

# deploy the application with the name "controls/checkbox" and print its URL
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

app = ds.app()  # create an instance of the application


# an utility function that loads the data
def get_data():
    return px.data.gapminder()


# a handler that updates the plot output based on the selected year
def output_handler(self, year):
    value = year.value()  # the selected year
    self.data = px.scatter(get_data().query("year==" + str(value)), x="gdpPercap", y="lifeExp",
                           size="pop", color="country", hover_name="country", log_x=True, size_max=60)


# a slider control that prompts to select a year
slider = app.slider(values=get_data()["year"].unique().tolist())

# an output control that shows the chart
app.output(handler=output_handler, depends=[slider])

# deploy the application with the name "controls/" and print its URL
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

The `Uploader` control can be used to upload single or multple files and access their content. Here's an example:

```python
import dstack as ds
import pandas as pd

app = ds.app()  # create an instance of the application


# a handler that loads a dataframe from the content of the uploaded CSV file and passes it to the output
def app_handler(self, uploader):
    if len(uploader.uploads) > 0:
        with uploader.uploads[0].open() as f:
            self.label = uploader.uploads[0].file_name
            self.data = pd.read_csv(f).head(100)
    else:
        self.label = "No file selected"
        self.data = None


# a file uploader control
uploader = app.uploader(label="Select a CSV file")

# an output control that shows the content of the uploaded file
app.output(handler=app_handler, depends=[uploader])

# deploy the application with the name "controls/select" and print its URL
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

The `Markdown` control can used to display a markdown text. Here's the same example that we used in for the `Input` control:

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

app = ds.app()  # create an instance of the application

# a markdown output
app.markdown(text="Hello, **World!**")

# deploy the application with the name "markdown" and print its URL
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

**`TODO:`** `Add output description`

