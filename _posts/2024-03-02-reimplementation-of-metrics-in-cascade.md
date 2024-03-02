---
layout: post
title: How I (Re)implemented Metrics in Cascade
date: 2024-03-02
categories: posts en
tags:
  - cascade
  - mlops
---

## Foreword
Not so long ago [Cascade](https://oxid15.github.io/projects/en/cascade.html) v0.13.0 came out. This was a release full of updates. Most of them retained compatibility with v0.12 but some of them were breaking. One of the examples are metrics. They underwent a serious change, which became the basis and inspiration for this post. Here I want to share my experience in building such a system of metrics for experiment tracking.

If you don't know what Cascade is - it is an MLOps library built specifically for small-scale ML projects. I wrote it myself for my own projects at work and continue to develop and conceptualize how ML development should be organized on different maturity levels. If you want to learn more about it, check out its [project page](https://oxid15.github.io/projects/en/cascade.html) and [GitHub](https://github.com/Oxid15/cascade). 

## Why
In v0.13.0 metrics in Cascade are now handled in completely different way. Here I will describe what was the state of metrics and why I decided to change it.

Metrics are very important part of any ML-project since they are the way we understand the quality of our artifacts and also monitor their quality as the time passes. Taking this into account, metrics were always the important part of Cascade. The term itself is used loosely - loss function values can be recorded as a metric even though separating targets from metrics it is a right thing to do. In Cascade metrics serve for tracking properties, not optimization so this looseness seems reasonable.

Metrics in old cascade versions stayed the same since very first releases. The idea was simple: let the Model have a simple arbitrary Python `dict` named `metrics` and let users fill it with basically anything they want to record. What metrics are, how and when they are computed is on the user's side. The library just does metadata management and doesn't care.
I imagined it like this:

```python
>>> model.metrics
```

```js
{
	"acc": 0.95,
	"precision": 0.98,
	"recall": 0.93
}
```

This seemed to be a great solution for several reasons: it is insanely simple and allows for a great amount of flexibility. However, this approach has a number of drawbacks that were the main reason for an API change.

Basically metrics became too wild. The amount of flexibility was too high and the price for it is the complex and undetermined visualization and analysis procedure for example. When designing visualization tools like tables or scatter plots I couldn't simply rely on some structure of metrics - I did not have a defined schema, a contract between the user that writes their metrics and me, accepting them for visualization.
The fact that metrics could be filled with anything appeared to be great and frightening at the same time. Metrics could not only come in any format, but also could be misused. For example user could fill a list with values trying to record a series of metrics instead of using a Line of models.

```js
{
	"metrics": {
		"loss": [0.5, 0.4, 0.3, ...] // This is wrong
	}
}
```

Aside from inconsistent schema, I encountered the other issue in my own practice. I didn't think about that when I implemented metrics for the first time, but *metrics have metadata too*.
This fact was not obvious to me until I had a case with many different validation datasets and metrics which made my `metrics` dictionary look like the following:

```js
{
	"metrics": {
		"main_train_ds_acc": 0.8,
		"main_val_ds_acc": 0.8,
		"secondary_train_ds_acc": 0.7,
		"secondary_val_ds_acc": 0.7,
		"main_train_ds_f1": 0.7,
		...
	}
}
```

Those long keys contained the information about the dataset and the split metric was computed on. Let's say this is not the way I would like to store that information.

All those issues led to the decision to design metrics starting from the basics.
## New metrics principles
In the next section I list the principles based on which I built new Cascade's metrics system.
### Metrics are values
... and also scalars. This was the most difficult decision to make, but I decided to restrict metric values to scalars. This reduces the flexibility greatly, but also possibility of errors or misuse. I listed some reasons why someone could have non-scalar types in a metric and all of them appeared to be wrong usage.
1. If you have a list, then use Line instead if it is a series of experiments
2. If it is a metric in a form of a list like ROC-curve values, then save them as a file (image or `.npy`) and link to the model using `add_file`
3. If it is a dictionary, then it could be metrics computed on different datasets - just use several metrics then

Based on this and my previous implementation I inferred the first principle - metric is a value. It should store its value as the most important component.
### Metrics are functions
This is new aspect that Cascade's metrics didn't have until this moment. Basically I say that metrics now not only store their values, but also can know how to compute them. This seems reasonable since metrics in other libraries such as `sklearn` already implemented as functions. It is also convenient too - we could have the same `Metric` object that will abstract details of computations from us and store the value at the same time!
In this setting the user has three possible ways to use metric: compute it somewhere else and pass the value. This is very similar approach to that we already had.  Just pass the function for Cascade to do the rest or to extend `Metric` class in order to customize and add their own metric.

```python
# Metric as a value

model = BasicModel()
model.add_metric("acc", 0.66)
# This creates Metric object under the hood anyway
```

```python
# Metric as a function

def accuracy_but_fn(gt, pred):
	return sum(gt == pred) / len(gt)
```

```python
# Metric as a class

class AccMetric(Metric):
    """
    This is accuracy
    """
    name = "acc_but_class"

    def compute(target, pred):
        self.value = sum(target == pred) / len(target)

model.evaluate(x, y, metrics=[AccMetric(), accuracy_but_fn])
```

This approach allows for certain amount of backwards compatibility since metrics still can be computed externally and also facilitates thoughtful design: users are encouraged to create classes for their metrics which then can be centralized, tested and documented properly.
So this is "metrics are functions". This principle helps to standardize metric computation since evaluate can accept objects of class `Metric` that already knows how to compute its value.
### Metrics have metadata
I already mentioned this in terms of knowing which dataset and split some value was obtained even when metric stayed the same. This calls for a set of default fields for metric that users can fill in order to get deeper understanding of their model, when evaluating it on different datasets, splits, etc.
Following is the list of fields I decided to include in the default metric class:

```python
name: str 
# Name of the metric

value: Optional[MetricType] 
# Scalar value of the metric, by default None

dataset : Optional[str]
# Dataset on which metric was computed, by default None

split : Optional[str]
# The split of the dataset for example train or test, by default None

direction : Literal["up", "down", None]
# Is metric better when it is greater or less, by default None

interval : Optional[Tuple[MetricType, MetricType]]
# Upper and lower boundaries of value, by default None

extra : Optional[Dict[str, MetricType]]
# Extra values that needs to be stored with metric, by default None
```

`name` and `value` are more or less obvious. Datasets are now of the type `str` since I expect users to only fill the name, but in the future I plan to introduce proper dataset links into this. Although `split` as a string seems totally okay for me. Direction is an important value when it comes to ordering models by the most performant: some metrics directed up and better models are with greater values of metrics like F1 score or accuracy, but some metrics are better when minimized since they represent unwanted properties such as any loss or RMSE. Interval is great when measuring metrics with confidence intervals, and it is needed to record upper and lower boundaries.
There is also `extra` parameter which was specially reserved for several reasons. First is backwards compatibility - if scalars are now required, old metrics may not fit into this system, but I would like to still have access to my old repos. Having this field allows inserting everything that does not fit there. The same ability is the second reason: I don't know if I included enough default fields and maybe for some cases some additional information will be needed. This field then works as a trade-off between old solution's flexibility and requirements for analysis and visualization: you still can save anything in `extra` but Cascade in turn does not work with this data since the schema for it does not exist.
This was the latest principle: metrics have metadata.
## Conclusions
In conclusion I would like to summarize, give the whole picture of how metrics were implemented, and how they will progress further.

Following is just a pseudo-python just to give an idea of an interface, not how it is [actually implemented](https://github.com/Oxid15/cascade/blob/105f60648d68482d31cc9b6bfd83bc6b72b83a35/cascade/metrics/metric.py).

```python
class Metric:
	name="metric"
	value=None
	dataset=None
	split=None
	direction=None
	interval=None
	extra=None

	# Default method for obtaining a value
	def compute(self, *args, **kwargs) -> MetricType:
		# self.value = ...
		# return self.value
		...

	# This one is for additive computation
	# When you cannot compute metric on the whole dataset
	# this will allow to compute it in batches
	def compute_add(self, *args, **kwargs) -> MetricType:
		...

	# For serialization purposes
	def to_dict(self) -> Dict[str, Any]:
		...
```

As you can see metrics are now complex and standardized objects that follow principles stated above: they store their values, know how to compute themselves and have additional metadata.

But how metrics are stored in the model in this case? I decided to store it as a list of Metric objects now. Users can link metrics to the model using the following:

```python
model.add_metric("name", value)
model.add_metric(metric_object)
```

If the metric was added as an object or a value, it is not computed. If you want to define metrics and then automatically compute and add them, then call `evaluate()`.

```python
model.evaluate(gt, pred, metrics=[Metric(), metric_function])
```

This will obtain all the values and then create metrics by calling `add_metric` internally.

The way metrics were redesigned and implemented is suggesting a further progress of this module. My idea is to create a library of default metrics inside Cascade to simplify the integration of external metrics by filling all the default fields and documentation properly. For now I have implemented only several basic instances, but I plan to continue expanding the library of default metrics.
You can use default metrics like following:

```python
from cascade.metrics import Loss
from cascade.metrics.classification import Accuracy
```

However, since I implemented only basic ones and those are obviously not enough, I added an interface for `sklearn.metrics` module that integrates scikit-learn metrics into Cascade.

You can just use the name of the metric from `sklearn.metrics` inside special class like this:

```python
from cascade.utils.sklearn import SkMetric

metric = SkMetric("f1_score")
value = metric.compute([0, 1], [1, 1])
# 0.6666666666666666
```
## Migration

Since metrics now function in a very different way, some measures to ensure (more or less) smooth migration from previous version were taken.
There are two points of migration - code that uses metrics and Cascade's repos on disk.
Previously code usually looked like this:
```python
model.metrics.update({"rmse": 1.3, "mae": 0.4})
```

To migrate we should change this to:
```python
model.add_metric("rmse", 1.3)
model.add_metric("mae", 0.4)
```

Previous evaluation code could look like this:
```python
model.evaluate(x, y, metrics_dict={"acc": accuracy_score})
```

However, v0.13.0 way of doing this is:
```python
model.evaluate(x, y, metrics=[Accuracy()])
```

If you try to use new version on old repos, it will detect it and fail not to break anything. If that happened you need to migrate your repo to the latest version.

Migration of model repos can be made automatically by using Cascade CLI
Just locate repo folder and use this command. This will automatically make the repo compatible with the latest version.

```bash
cascade migrate
```

This tool was tested to migrate from 0.12.0 to 0.13.0 and may not work correctly for older versions.

## Final thoughts

I am always excited to find new ways of making the work of ML engineers easier, by developing MLOps solutions such as Cascade. I hope that standardizing metrics will be an advancement for this project and boost the productivity of Cascade's users.