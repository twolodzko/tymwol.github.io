---
layout: post
title:  "Can Machine Learning be Lean?"
date:   2021-05-27
---

Lean started with [Toyota Production System][toyota], followed by [lean software development][lean-software]
and [lean startup][lean-startup]. The key take-away of lean is cutting-off the unnecessary processes and
leaving only the ones that bring the value to the customer, as Marie Condo would say, the ones that 
"spark the joy". The lean ideas were translated to software development, but could the ideas be used
in machine learning and data science? Let's walk through the [seven deadly wastes][wastes] and see if
they apply to machine learning.

# 1. Waste of defects

Same as with regular software development, a lot of defects can be avoided with testing the code.
[Jeremy Jordan][jj-testing-ml], [Luigi Patruno][mlp-testing-ml], and [Martin Fowler's blog][cd4ml]
are nice resources on testing machine learning models. The obvious tests are unit tests for
the code, or verifying the test error metrics, but among less obvious solutions you can run smoke 
tests (run the whole pipeline on simplified data, with reduced settings to see if nothing "smokes",
to catch the problems early), or having test data cases for which the model [needs to make][karpathy]
correct predictions.

# 2. Waste of inventory

In traditional software engineering waste of inventory is usually understood in categories
as partially done, that doesn't bring much value. While those issues apply to data science as well, there are
also examples of waste of inventory that are specific for data science. The obvious ones
are idle jobs, like virtual machines that were not closed after finishing the computations.
The less obvious ones may be using inadequate or wasteful technology solutions, for example, 
unnecessarily [using grid search][grid-search] for hyperparameter tuning, while random search
or Bayesian optimization may be a better choices, or using big data technologies (Spark) when
they are not really needed because the dataset can fit the memory of a single machine
([Pafka Szilard][rf-benchmark] has shown that Spark's implementation of random forest can be
less efficient than the regular implementations). Training a model that cannot be used in production enviroment
(for example, it is too slow, or does not match the memory constraints) is also a waste, so
such issues should be considered in advance.

# 3. Waste of processing

The classic example of waste of processing in software engineering are unnecessary processes. 
In data science, this can be meetings. [Doug Rose][agile-ds], as several others, have noticed
that Scrum does not work well for data science teams, because of problems with planning and
deliverables, that are usually less obvious and hard to define. Scrum has a number of team
events that may be considered as wasteful if they do not bring much value for the team.

Another example of unnecessary processing are repeatable steps (getting the data, cleaning it,
feature engineering, etc) that could be automated. Documenting those steps, writing Bash scripts,
or Makefiles to automate them may be easy solutions to avoid the waste.

# 4. Waste of waiting

Waiting until the model finishes training is probably the biggest waste of waiting in machine
learning. Unfortunately, it is also the one that is the hardest to avoid. To speed up the
training you could always use more powerful machine to do the computations. Such machines
are more expensive, but the question to ask yourself if it is really cheaper to use a
cheaper virtual machine, if this would lead to longer model training, if you consider the
hourly wage you pay your data scientist waiting for the results?

Waste of waiting is also likely reason for popularity of frameworks such as PyTorch as compared
to TensorFlow 1.x. While PyTorch enabled data scientists to interactively interact with the code,
TensorFlow 1.x translated the code to computational graph, that wasn't that easy to work with.
Working with PyTorch was more direct, hence faster, so TensorFlow 2.x decided to introduce the 
eager mode. 

....[ml-powered] describes how building a machine learning based product can be done in
an iterative fashion, where we try as to release it often to gather the feedback from the
users. 

# 5. Waste of motion

Starting a data science project often needs a considerable amount of legwork (or Zoom-work). 
You need to meet the stakeholders, sometimes the potential customers or domain experts to
learn more about the problem, the data owners to learn how to access the data, etc. Those
are the obvious areas of improvement in terms of waste of motion.

The technological side of the waste of motion is usually solved by automating the data and
machine learning pipelines using technologies like Bash scripts, Airflow or Luigi pipelines,
that take care about the moving parts of the process by themselves. Shared repository and
version control helps to keep all the scripts and notebooks in a single place, so they
don't need to be chased over your or your teammates machines each time they are needed.

# 6. Waste of transportation

In software engineering [task switching][multitasking] is considered as an analogue of 
waste of transportation. In data science, additionally we can consider moving the data
between our workers and databases in terms of waste of transportation. One of the solutions 
is using feature stores with precomputed features instead of downloading a larger, raw dataset
and computing the features on-the-fly.

# 7. Waste of overproduction

Adding an extra features to the software is a waste of overproduction. Machine learning models
usually do one thing only, so except producing unnecessary supporting code, it is not that big
issue. The example of waste of overproduction in machine learning could be using an overtly 
complicated model to solve a problem that can be solved using a simpler solution. Many natural
language processing problems can be solved using solutions such simple as TF-IDF features, or
regular expressions, rather than the state-of-the-art language models with millions of parameters.
Starting with a simple model (rule-based, logistic regression, decision tree) may be a good start
to preventing overproduction. The simple model may turn out to be good enough, at worst it can
be used as a benchmark when developing a more complicated model. As described by ....[ml-powered],
starting with a simple model is also a chance to build the supporting infrastructure fast and
it will serve as a minimum viable product enabling you to get the feedback from the potential
users of the product early.

# Conclusion

Are we there yet? Data science and machine learning seem to be behind of software engineering
in terms of agile and lean processes to making them more efficient, but the MLOps movement ..... 


 [wastes]: https://www.linkedin.com/learning/dealing-with-the-seven-deadly-wastes/seven-wastes-overview
 [lean-software]: https://www.goodreads.com/book/show/194338.Lean_Software_Development
 [lean-startup]: https://www.goodreads.com/book/show/10127019-the-lean-startup
 [jj-testing-ml]: https://www.jeremyjordan.me/testing-ml/
 [mlp-testing-ml]: https://mlinproduction.com/testing-machine-learning-models-deployment-series-07/
 [cd4ml]: https://martinfowler.com/articles/cd4ml.htmls
 [karpathy]: https://www.youtube.com/watch?v=hx7BXih7zx8
 [mlops-book]: https://pages.dataiku.com/oreilly-introducing-mlops
 [ml-powered]: https://www.goodreads.com/book/show/50204636-building-machine-learning-powered-applications
 [agile-ds]: https://www.linkedin.com/learning/learning-data-science-using-agile-methodology/welcome
 [toyota]: https://en.wikipedia.org/wiki/Toyota_Production_System
 [rf-benchmarks]: https://github.com/szilard/benchm-ml
 [multitasking]: https://www.apa.org/research/action/multitask
 [grid-search]: https://stats.stackexchange.com/questions/160479/practical-hyperparameter-optimization-random-vs-grid-search
