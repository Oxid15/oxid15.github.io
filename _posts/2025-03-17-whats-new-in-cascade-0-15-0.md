---
layout: post
title: What's new in Cascade 0.15.0
date: 2025-03-17
categories: posts en
description: "The new 0.15.0 Cascade release highlights"
tags:
  - cascade
  - mlops
  - release_notes
---

## What's new in Cascade 0.15.0

I started to work on this release immediately after I released previous - 0.14.0. It was (oh my) almost half a year ago. Having a full time on-site takes its toll I guess. Anyway, I am really excited about this release as it opens a whole new variety of possibilities for future development. This post is about those. I will shortly explain what's inside with useful links for the ones wanting to go deeper.

You can find the PR with the changelog [here](https://github.com/Oxid15/cascade/pull/271). However, here is the summary:

- Additions
	- Configuration management
	- CLI-based results query language
	- Confirmation before deleting artifacts
- Enhancements
	- Removed dependency on flatten_dict
	- Fixed `__init__` call bug
	- Handle removed models and lines
- Removes
	- Old history logging stuff, old validators, repo concatenations

## Configuration management

I have a whole separate post about this coming soon, but I will reiterate here.
ML heavily relies on configuration. Even in simplest cases there are always some hyperparameters you need to tweak. Projects that live long enough can have enormous configs which become a declarative language themselves.
Given this, I decided to Implement Cascade-side of this configuration problem. The existing system is twofold: there is `Config` class and additional support for it in `Models` and also `cascade run` command.

`Config` allows you to wrap your existing experiment configs in Cascade-compatible object and `cascade run` allows you dynamically set the values in this class **without changing the code**.
In reality it looks even more like magic than it sounds. Let me give you an example.

Suppose you have `script.py` like this. In my practice a lot of training proof-of-concept scripts end up with a config like this, in global variables.

```python
from cascade.models import BasicModel
from cascade.repos import Repo

LR = 1e-4
BS = 32

if __name__ == "__main__":
    model = BasicModel()

    model.params.update(
        {
            "lr": LR,
            "bs": bs,
        }
    )

    # Here we train model

    repo = Repo("test")
    line = repo.add_line()
    line.save(model)
```

Then we run it like this.

```bash
python script.py
```

To run a new experiment you will need to update this code and run it. If you want to run multiple experiments at once or automate this, it will be more difficult.

Here is how you update it to get configuration management for free.

```python
from cascade.base import Config
from cascade.models import BasicModel
from cascade.repos import Repo

class CFG(Config):
	lr = 1e-4
	bs = 32

if __name__ == "__main__":
	model = BasicModel()
	model.params.update(
		{
			"lr": CFG.lr,
			"bs": CFG.bs,
		}
	)
	model.add_config()

	# Here we train model

	repo = Repo("test")
	line = repo.add_line()
	line.save(model)
```

To modify config without editing the source run it like this.

```bash
cascade run script.py --lr '1e-6' --bs 64
```

You should see:

```text
You are about to run script.py
The config is:
{'bs': 32, 'lr': 0.0001}
The arguments you passed:
{'bs': 64, 'lr': 1e-06}
Confirm? [y/N]: y
```

After running this you have:

1. The original script that remains unchanged
2. The new config magically tracked with `model.add_config()` inside `files` folder
3. Logs tracking that you get for free by adding `--log` to the command, log will be added to the same `files` folder
4. Exception handling is also included, logs and configs will be saved in a temporary directory

See [this tutorial](https://oxid15.github.io/cascade/en/v0.15.0/tutorials/advanced_experiment_management.html) for more formal explanation and this post for how this works under the hood.

## Results querying

When you have a lot of experiment results it is hard to analyze many experiments by looking at `meta.json` files. You can use Viewers, but not every small task can be solved using them, they lack flexibility.

This is why querying was created. For quick analyses of results you can write a CLI one-liner now. Cascade CLI now has small query language of its own.

For example if you want to check some experiment and you have its slug (unreal situation, but bare with me for this simple example).

```bash
cascade query slug created_at filter 'slug == "brave_solemn_urchin"'
```

The syntax is simple - you write `cascade query` and then a list of columns without commas. The language supports columns as (limited) Python expressions. Then you can write a `filter` expression like in the example below using the fields found in meta.

You can also sort your results using keyword `sort`. Use `desc` to change default ascending order of sort.
Here is how to get top-5 experiments by accuracy:

```bash
cascade query slug created_at sort '[m for m in metrics if m.name == "acc"][0].value' limit 5
```

Here we also used `limit` keyword to get only 5 results. There is `offset` too.

You can find full tutorial on Cascade CLI querying [here](https://oxid15.github.io/cascade/en/v0.15.0/tutorials/results_querying.html) and howto [here](https://oxid15.github.io/cascade/en/v0.15.0/howtos/queries.html).

## Confirm before deleting artifacts

In Cascade CLI you can delete artifacts using `cascade artifact rm`. From now on it will require `-y` or manual confirmation to remove stuff. This is obviously a safer and more responsible option.

## Goodbye flatten_dict

Cascade used `flatten_dict` package internally and required it as a dependency. Since it is relatively easy to flat dicts and the package is pretty much unmaintained, it was decided to remove this dependency by implementing the same functionalities internally.
Thanks to the effort of contributors it is now done!
One of Cascade goals is to be lightweight without loads of dependencies, this decision was aligned with that.

## Bugfix

For now Cascade supports if the user does not call `__init__` in their `Dataset` subclass, so this fix aims to continue that. However, this may be considered for changes in the future.

## Removes in the middle problem

Previously if you had a Repo with 4 lines like this:

```text
00000
00001
00002
00003
```

And you removed the line in the middle.

```text
00000
00001
00003
```

When you run `repo.add_line()` again, the algorithm would create `00002` (because of how it was implemented, don't ask...).

Now this is over, the new line (or model) naming algorithm is now smart and will create the line `00004` instead.

## Removed stuff

In `0.14.0` lots of stuff was planned to be removed with deprecation messages. (To be fair, I almost forgot to do this and noticed those messages by accident.)
Some of the removed entities include:

- `DataRegistrator` and `HistoryHandler` are replaced with `DataLines`
- `Validators` - are replaced with `SchemaModifiers`
- `ModelRepoConcatenator` is replaced with `Workspaces`
- `HistoryDiffViewer` will not work without `HistoryHandler`

Cascade still keeps old `ModelRepo`, `ModelLine`, etc classes for backward compatibility and smoother adoption, but will also be removed later.

## Resume

Release `0.15.0` is a huge one in terms of new features. I consider it one of the most responsible ones - I did a deep dive for each feature and wrote an extensive set of tests. Hope you'll like it! You can fill issues on [GitHub](https://github.com/Oxid15/cascade), write me an email or mention [Cascade account on X](https://x.com/cascade_mlops).
