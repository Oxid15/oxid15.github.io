---
layout: post
title:  "Cascade project"
date:   2022-07-22 00:00:00 +0000
categories: posts en
tags: cascade mlops
---

Lightweight ML Engineering Library

## Foreword
This post is on the project that I was developing for 6 months now. It is named Cascade. ML engineering library for fast-paced ML development, management of metadata and artifacts.
I write this to make reader familiar with Cascade as a project, to make first attempt to build community, find people that may be interested in the usage and development of such tool.

[Documentation](https://oxid15.github.io/cascade/en/latest/) | [Discussion](https://github.com/Oxid15/cascade/discussions) | [Code](https://github.com/Oxid15/cascade)

How it started
The idea of this kind of tool was in the air for several years since I get my first position as an intern in ML. I witnessed how the problems of unmanaged ML development are solved (tough I wasn't aware of them) on the scale of not so small company. What are those problems?

Let's consider following setting. You are ML-researcher/engineer in the small company or (what is more common) individual. You run a lot of ML experiments in your Jupyter-notebook, training your models with different hyperparameters and data with no structure. You just quickly write your data pipeline and trainloop with evaluation saving the models and their metrics manually on disk.

Eventually you end up with a set of binary artifacts (your ML-models) on disk in some arbitrarily named folders like experiment228/00037/00037.pkl etc. Suppose VCS was used to track changes in the code producing those models. It is not always true especially in the case of small classical models when training may take seconds and track each change in the parameters by hands will be too time-consuming. Even with the presence of code versions we cannot tell using them which version produced which model that is stored on disk at the end. Suppose we wrote all metrics needed in metadata to the same folder and can choose model using those metrics, but if we would want to go back in model versions and continue from there, we couldn't do this.

In this setting we are condemned to fail when the number of different experiments becomes too large. All automatic management that can be done by us to prevent or postpone our failure is made by hands and needs to be repeated in the following similar projects. These are inevitable metadata issues. Aside from them there are also reusability issues. We have some arbitrarily written data pipeline and trainloop. Without any modularity or isolation in this code we cannot reuse it in similar projects without restructuring.

Some of these issues in the company I worked were addressed using internally developed tool, which then inspired me to create my own. So how it started?

It was pretty classical project - I was supposed to train a classifier to use it in the downstream product tasks. The dataset was not so large so I choosed to use classical algorithms for this task. After preparations I ended up with some boilerplate train.py which contained all data preparation steps and training loop where different classifier models were trained and evaluated. In the end results were compared and I was able to choose the best model, that was saved.

Two problems were apparent at this stage: a) the data come from many different sources - the processing of different datasets has some different parts and some common, the code is duplicated and b) not all models are saved, experiments are not tracked in any way.

At this point I decided to address these problems by creating utilities to use in this project - Dataset and Model classes.
Dataset was the wrapper around sequence of data points. It allowed me to simplify the code of different data pipelines. I used Dataset to load data from disk and applied a series of Modifiers to form a pipeline. Repeating stages of data processing were now incapsulated and could be used without repetition and different stages could easily be identified.
Model class was a wrapper around inference and additionally allowed me to define how the model is saved and loaded on disk.

Cascade started as utils that make life easier inspired by great ideas that are in the air last years.


## Why it is used
There are a number of good solutions for problems mentioned. The main reason they were not suitable for my purposes it that they are designed for big companies. When you are an individual developer or researcher or a part of a small team, it is difficult to use such frameworks. The other reasons include inability to quickly prototype and write reusable data pipelines.

Considering this it was obvious that some another solution needed. That will be suitable for small teams, more flexible and with the ability to use only some modules when needed.

Cascade for me became such kind of solution for a number of projects at work and some research projects at university. I hope that it will also benefit other people experiencing similar issues and want to change and develop the tool in the way that will be more suitable not only for me.

## How it looks now

What can you do now using Cascade (0.5.2)? A number of things that include:
- Build data processing pipelines using isolated reusable blocks
- Easily get and save metadata about this pipeline with little or no additional code
- Easily store model's artifacts and all model's metadata
- Use Web UI tools to view model's metadata and metrics to choose the best one
- Build data validation blocks to ensure quality of data that comes in the model
- Use growing library of Datasets and Models in `utils` module that propose some task-specific datasets (like TimeSeriesDataset) or framework-specific models (like SkModel)

Now 13 releases since 0.1.0 Cascade is more mature and considered project. It has some ideology and [basic concepts](https://oxid15.github.io/cascade/en/latest/concepts.html) that are still developing of course. Since I actively use it in other projects it has certain responsibility as a dependency for them. It makes me think about legacy and how each change will affect old projects. This is important even though Cascade is not 1.x.x yet.

## Where it is going
Cascade is only at the start of its way and the directions are broad and vague. I list my toughts on how the development will be held in the following section. However, I hope that with the help of community the picture can become more clear. So what are the directions?

### Advanced metadata analysis
When doing ML-project with the help of Cascade many metadata files with parameters, metrics and dataset descriptions are written. Though they are human-readable, in most cases they shouldn't be read by a human. Cascade already have solutions for this in form of different [Viewers](https://oxid15.github.io/cascade/en/latest/concepts.html#viewers).
HistoryViewer helps to see history of changes in Model's parameters. It automatically figures out the tree of changes made in the parameters and shows it to the user in respect to the given metric. At this moment it is in raw condition and needs **improvement and feedback**.

**MetricViewer** is more broadly used tool. It helps to easily see metrics of models saved and choose the best one. It presents metrics in the form of the table either in console or as interactive web table with plots. Due to its intensive usage it is more or less debugged. Its **interface will be extended** for more fine-grained metric and parameters analysis and easy model selection. Additionally **Web UI should become more advanced** allowing for more interactivity.

Another problem that is present is **model files management**. After intensive training experiments it comes time to eventually free disk space from redundant models. Cascade to this moment does not allow to manage model files. I consider this as a direction for further development.

## Pipelines

Pipelines are the second focus of Cascade. And by pipelines I mean not only data processing pipelines, but also **model pipelines**, which are now developing with the introduction of ModelModifier.
To build advanced pipelines with some task-specific or framework-specific blocks cascade.utils is used. Utils is planned to be extended to potentially become separate package. It also needs more extensive testing and documentation, which is now absent.
Data validation also needs to be further developed to perform automatic data drift checks or NaN checks for example.

## Concepts

Some more difficult to implement ideas include code tracking - to dump all the code that was used to produce some model into this model metadata, meta loading - to load existing metadata of an object from disk if found and pipeline visualization in form of graph to easily see its structure and read meta of all block separately.

## First major version

Of course Cascade pursuits version 1.0.0 when more responsibility for legacy will be taken and API will be polished, tested and documented enough to be fixated until next major release.

## PyPI publication

It doesn't mean that now you can't install Cascade using pip. You can, but using the link from Github which is not so elegant and convenient. I have attempted PyPI publication. First problem was name conflict with some old Github project also named cascade, so I eventually published the package using the name cascade-ml, but there was another problem for which I didn't found a solution. So by now it cannot be installed using the name, but it will be soon, when I figure out the issue.

## Closing word

There was Cascade - ML engineering library and toolkit for small teams and individuals, developers or researchers. I am open to any discussion regarding the project, its architecture, features and future directions and hope that it would be useful not only for my specific tasks, but also for others!

[Documentation](https://oxid15.github.io/cascade/en/latest/) | [Discussion](https://github.com/Oxid15/cascade/discussions) | [Code](https://github.com/Oxid15/cascade)
