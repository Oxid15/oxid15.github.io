---
layout: post
title: Configuration Management with Cascade
date: 2025-02-13
categories: posts en
description: "The reasoning behind new Cascade feature"
tags:
  - cascade
  - mlops
  - configuration_management
---

## Motivation

In the machine learning world configuration is everywhere.

Most experiments are made with configuration change. Those are usually experiments with parameter tuning - you change something and see the result. Although configs can be more powerful if made like feature flags, when setting some values can change the executed code a lot. We will left the problem of the overuse of feature flags out of the scope of this post though.

This post is about why and how configuration management became a part of Cascade in new [0.15.0 release](https://oxid15.github.io/posts/en/2025/03/17/whats-new-in-cascade-0-15-0.html).

## Design

After deciding that Cascade needs something to work with configs I did small overview to see what others already have. Luckily I stumbled upon some great features in projects either dedicated to configs or to experiment management. My overview does not try to be comprehensive or scientific - its purpose was to inspire me and see what already works for the users of other projects.

[Sacred](https://github.com/IDSIA/sacred) is an experiment management tool. Although without specific focus on ML, its decorators allow tracking every parameter that was passed to a function.
Although decorators are powerful, they inflict additional dependencies on experiment code, potentially making it less transfetable as [pointed out](https://www-guildai.netlify.app/faq/) by the creator of guildai.
I really liked their environment tracking feature, that allows tracking all python packages versions in the current environment. This should greatly contribute to reproducibility.

[GuildAI](https://github.com/guildai/guildai) is a tool with another paradigm. The problem with wrapping everything in trackers is not in possible speed overhead, but in dependency overhead.
By including tracking code into our clean training scripts we can strongly impede sharing of the code. Other person, wanting to reproduce our results, can have the same script and install its dependencies, but have troubles setting up their tracking API keys, databases and so on.
GuildAI provides very uninvasive way to interact with experiments using CLI. The feature I personally found cool and almost magical is parameter overrides. You write your script as usual, having your configuration in global variables (I do this kind of configuration a lot) and then you can run it with guildai, but say that for example "now learning_rate is 1e-3 instead of 1e-4". It will not change the code in your repo, but will make the requested changes in its own copy of it which it will end up running.
I think this feature enables very flexible experiments without having to bother your version control system.

[Hydra](https://github.com/facebookresearch/hydra) is a tool from Meta, that I found relevant for this case. It is very powerful config management solution.
What I found very useful is having a base config along with your new "override config". You write a small file with your overrides only which inherits everything else from the base config. In projects with large configs this should be very effective and especially in research. When you are doing an experiment, usually you want your change to be as small as possible to see the effect free of confounders. From the config's point of view this usually means a very small change. And when your config is large enough it can be painful to analyze what was changed and what was the baseline.
Another thing I found convenient is multiruns. Basically it is a built in grid search capabilities. In your config you can set ranges and it will loop through them. It is not complex or original functionality, but it should simplify things for the user.
Hydra also features log capturing
Comparing to what I decided to implement Hydra is too complex. This is mostly a scope mismatch - obviously it was built to solve problems at Meta's scale.

The tools mentioned have lots of other cool features that I didn't mention here. I would consider my overview intentionally superficial, so if I missed something it was by design.
Here is a list of things I found very useful and would like to see them as Cascade features.

- CLI config overrides without code modification
- Multiruns
- Reruns
- Log tracking

Not everything will become a part of a single release as I plan to change things incrementally and test them in real life scenarios.

## Configuration types

There are many ways one can configure a program. In my classification I decided to focus on ML scripts. I would divide ways of how you can configure your scripts into three categories: text files, configuration in code and command line arguments. This division does not mean that one job can only be configured with one type, although I would not try using them all at once for a simple job.

JSON, XML, YAML and the usage of other similar formats I would classify as configuration in text files. It can be very useful, since you can juggle prewritten files and change the program behavior with them. They are useful when your program has several modes of function, so that you can switch configs and run it differently. However, when you experiment quickly it becomes harder to track those files and when they are large, it is easy to make mistakes.

Configuration in code is usually done with global variables in scripts, or using special config objects. This type of configuration is very flexible, since you can use the power of programming language to preprocess, validate or create templates. It is also easier to switch to this type if you do not have configuration for your script. It is okay for quick experiments, but only in case when you do not have to run experiments in parallel. Tracking becomes harder - if you track your script with version control software you have to track your configuration values once in a while, which may not be okay for quick experiments and interfere with tracking of code changes.

Command line arguments stand out, since usually there is no file where the config values are stored. The values are only passed when the script is called and are not saved by default. Although unusual, I think this is a valid type of configuration and a very popular one also. With command line you can experiment freely and do not have to track your configs in VSC, but tracking becomes the main problem then. Since values live only while command is running, some tracking solution needed.

Having all types reviewed I decided that eventually Cascade should work somehow with every type of config, but the first step needs to be small.

## Implementation

As the first set of features I decided to do:

- Configs in source code
- CLI overrides
- Config tracking
- Log tracking

Those seemed like a solid starting set of features.

### Configs in Source Code

I have chosen source code configs since they are the most effortless type. Those are the first configuration you will do if you have a single script. Personally, I put everything in global uppercase variables above `if name == "__main__"` block.

I needed configs to be integrated into existing Cascade system - make them trackable. The same can be achieved in different ways. I had an idea about treating configs differently from everything else - create special Lines for configs that would work like ModelLines, assign versions like in DataLines, etc.
Another idea was to link configs to the model somehow. Since single saved model is a result of an experiment, a point where we log results, we could store related configs there.
After some time I decided to chose the second option. It is easier, makes less new entities, does not change the focus of Cascade.
I implemented configs as simple containers for data. They do several things:
1. Integrate with Cascade with `to_dict` method
2. Wrap simple dicts
3. Integrate with `argparse`

This is approximately how I did it, tried to keep everything simple. The example has only init.

```python
class Config:
    def __init__(self, cfg: Union[Dict[str, Any], Namespace, None] = None):
        if isinstance(cfg, Namespace):
            cfg = vars(cfg)
        elif cfg is None:
            cfg = {}
        elif not isinstance(cfg, dict):
            raise TypeError(
                f"Argument of type {type(cfg)} passed."
                " You can initialize the Config only with dict or argparse.Namespace"
            )

        self.__dict__.update(cfg)
```

### CLI Overrides

The main problem with configuration inside source code is having several instances running in parallel. You need to change the global variables, run the script, make sure it started and then change variables again for the second run. In this case if the first run fails, you are going to need to revert your changes. It is easy to make a mistake in this and other more complex cases.

Those drawbacks can be mitigated by dynamic overrides. If we were able to set variables just for a single run without altering the source file for others it would help.

So I had several options. First was to interrupt main function somehow with a decorator or in some other way, parse and pass overrides. I didn't like this option as too invasive. The scripts with Cascade tracking should be able to run as if they don't have it.
This is why I followed the guildai idea of globals overrides. If I could edit the source code before running the script, then it would be easy to have several concurrent runs.
Guildai makes a copy of a whole project to run a script. This is a valid approach to have truly independent runs and trackable code, but I didn't want it for Cascade as it brings a lot of complexity with copying (what to copy, when, what if the files are too large, etc).
I decided to make changes to the source code on the fly for each run. In Python one can use standard tools to run code as a string. If I read the script manually, change it appropriately and then run, I never change the original source code and at the same time run independent versions of the script.

This is how it works in detail.

1. The user writes a command like `cascade run my_script.py --lr 1e-5 --batch_size 16`. This will invoke click command.
2. It will read the text from `my_script.py`
3. Then it finds the config class definition (any class that inherits Config) and parses the default values.
4. Then it starts to parse the overrides. I decided to parse args myself to not to worry about the formatting of everything, so the parser gives me everything that is not an argument of `run` command. I create a dictionary of values passed. Since everything in CLI command is a string, I needed a way to figure out types without manually creating a parser. It turned out Python already has a great tool for this, which is called `literal_eval` from `ast` module. It is very limited in what can be parsed, but has just the functionality I needed for this.
5. I parse the source code using the same `ast` module, which I found awesome. It represents the whole script in a nice looking syntax tree. Parser looks for values user passed to override and puts the values into the tree. When the tree is modified parser compiles it back to the source code and can run as a text script.
6. Script can have no config and no overrides and it is totally okay too!
7. It `pprint`s everything out and if `-y` flag was not passed, it asks for confirmation
8. Then special `CascadeRun` context manager kicks in. It handles the process spawning, errors and environment. It will create a temporary directory for the run and put useful stuff in it for future reference.

### Config and Log Tracking 

I already mentioned how configs were designed, but didn't get to the topic of tracking yet.
There are similarities in how configs and logs are tracked while running a script.
Since I decided to make it a part of a regular Cascade tracking routine it should be tracked as a part of model's files.

The main problem when trying to attach logs and configs to a model through cascade run is the lack of information. When you do `cascade run script.py` it does not know much about the code inside the script. It does not know when and where models are saved. The script may not have Lines and Models at all, only the config. At the same time the script itself does not know whether it is run by `cascade run` or by regular python call. When it saves models it does not know where the logs are.

After some thinking I found rather elegant solution. Since `run` spawns a new process it has full control of its environment variables. I thought that this could be the channel of communication between `run` and the script.
When script needs to save the logs and configs it can check if the locations are available in special environment variables. The good thing is that it could be done inside the library without the concern for the user. To use this feature and attach log and config files you just need to do the following.

```python
model.add_log()
model.add_config()
```

Usually when you attach files, you need to provide the path for them. In case of those methods, the paths are handled automatically using environment variables.
In case that we start the script without `cascade run`, those methods will raise a warning without interrupting the flow. This is very important, because it allows to use scripts without the need to do `cascade run` every time.

## Failure handling

What happens when the script finishes with an error? Logs and configs are saved by Lines when line.save is called. What if fail occures before this moment?
After script finishes (with or without error) the context manager mentioned should remove temporary files - configs and logs by default. And this was the case for my first implementation.
Cascade will keep configs and logs in the temporary directory .cascade/runs for the user to decide how to deal with them. Since each unique run has its own name for temporary folder, the path is printed as the part of the error message when script finished. This enables such important features as postmortem analysis of logs and configs that were used when the error occured.

## Summary

So at this moment with version `0.15.0` you can do the following things.

Run a single experiment with config overrides.

```python
from cascade.base import Config

class MyConfig(Config):
    a = 1


print(MyConfig.a)

```

```bash
cascade run script.py -y --a 0
```

Run an experiment with config and logs tracking.

```bash
cascade run script.py -y --a 0 --log
```

Run an experiment without config, just to track logs.

```bash
cascade run script.py -y --log
```

This set of features is simple, but already creates a powerful basis. Users can be in control of their configuration for different ML scripts, override and track them the way Cascade does tracking.

## What's next

The configuration management is the broad direction for development. The fact that there are many projects that do solely this task, supports this view.
I cannot be quite sure of the next steps given that Cascade is a side project and I have a full time job. However, there are things that I want to do, even if I don't know if they are possible in reasonable time.

### Reruns 

Sometimes you want to retry an experiment with the same set of params as the previous one. This may be a reproducibility run or just another attempt after fixing previously hidden bug. I don't have drafts of implementation yet, but I would like to see something like this in the interface:

```bash
cascade run --rerun 00000/00001 script.py
```

Or maybe even:

```bash
cascade rerun 00000/00001
```

### Inheritance

In some cases you do not want to repeat previous experiment exactly, but want to override your previous override. This may sound messy, but this is the natural way we do experiments - as branches of thoughts and incremental modifications.
This is where inheritance can be used. I like the way it is implemented in Meta's Hydra. The user has a base config and provides an override config with only the fields they've chosen to change.

### Multiruns

Automatic hyperparameter combination search is a part of multiple popular MLOps solutions at this point. Since Cascade offers the functionality of running a script with a config, why not pass the list of values or a range?
This is an easy extension of the same theme, but I would like to continue ot further and will think about how I can tie this to hyperparameter search engines like optuna for example. This could be an interesting development direction and further integration with other solutions.

### Other configuration types

Of course not everyone uses Python as a configuration language. In the future, other ways of configuration can be integrated too.
For example Cascade could analyze script's `--help` output and wrap its schema with overrides.

## Conclusion

In this post I explained the background and details of how I implemented config management in Cascade. If you want to start using this feature it is already available as a part of `0.15.0`, which you can get by running `pip install cascade-ml==0.15.0`.
Documentation is available [here](https://oxid15.github.io/cascade/en/latest/). Contact me if you have any questions/suggestions through email, [Cascade account on X](https://x.com/cascade_mlops) or [GitHub issues](https://github.com/Oxid15/cascade).
