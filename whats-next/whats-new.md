# What's New

### dstack 0.6.3, March 2021

* The update introduces the brand new API for building applications. The new API is much easier to use. Please check up the updated [Quickstart](../quickstart.md) and [Controls](../concepts/controls.md) pages to learn about the new API. 

{% hint style="warning" %}
Note, because of this change, the update is not backward compatible. If you update to the new version, you'll have to redeploy your applications using the new API. Have any problems or questions? Please write us to on the [Discord channel](https://discord.com/invite/8xfhEYa).
{% endhint %}

* The update brings support for basic layouts. Now, it's possible to arrange application controls using the grid layout mechanism, where you can specify how many columns and rows every control is taking. Read the `Layout` section of the [Controls](../concepts/controls.md#layout) page to learn more about how it works.
* Within control handlers, it's now possible to use the `tqdm` package to report the progress of the application. Please read the [Progress](../concepts/progress.md) page to learn more about how it works.
* The update adds support for `sklearn` [pipelines](https://scikit-learn.org/stable/modules/generated/sklearn.pipeline.Pipeline.html). Now, it's possible to push and pull entire pipelines with custom transformers as models. For an example of using this feature, please read the [tutorial](../tutorials/simple-application-with-scikit-learn-model.md).

### dstack 0.6.2, February 2021

* Bugfixes

### dstack 0.6.1, February 2021

* Applications now support multiple outputs. The signature of the dstack.app\(\) function has changed. Now it has `controls` and `outputs` arguments. Note, it's not yet possible to choose custom layouts. Currently, all outputs are displayed one under another. Watch [\#98](https://github.com/dstackai/dstack/issues/98) to track the progress on multiple layouts.
* The supported controls now include `dstack.controls.FileUploader` that allows uploading one or multiple files.
* The application now includes a `Reset` button.

### dstack 0.6.0, January 2021

* `Application` is a brand new type of stacks. It allows Python developers to quickly build and publish applications and ML models.
  * It supports [interactive controls](../concepts/controls.md) \(check-boxes, combo-boxes, text fields, sliders, etc\).
  * It supports [interactive outputs]() \(plots, tables, markdown\).
  * It supports the built-in [caching mechanism](../concepts/caching.md).
  * It supports managing application [dependencies](../concepts/dependencies.md) \(local modules and packages as well as dependencies to third-party PyPi libraries\).
  * It supports writing application [logs](../concepts/logging.md).
  * See [quickstart](../quickstart.md) and [tutorials](../tutorials/) for more details.
* In `Settings`, it is now possible to manage users.
* The `Sharing` options now include the following settings:
  * The stack can be accessed via its link by anyone \(`public`\)
  * The stack can be accessed via its link by registered users \(`internal`\)
  * The stack can be accessed by selected users only \(`private`\)
* `dstack` is now possible to run either locally via [`pip`](https://pypi.org/project/dstack/) or via [Docker](https://hub.docker.com/repository/docker/dstackai/dstack). _Warning, the conda and CRAN packages are not going to be supported anymore._
* [dstack.cloud](https://dstack.cloud) is the new version of the free cloud version of dstack that supports publishing `Applications` and `Models`. _Warning, the old version of the cloud_ [_dstack.ai_](https://dstack.ai) _is going to be discontinued on 15th of Feburary._
* The `Reports` and `Jobs` are not supported anymore.
* The `Stacks` page is now split into `Applications` and `Models` pages.
* The `Charts` and `Datasets` stack types are not supported anymore.

