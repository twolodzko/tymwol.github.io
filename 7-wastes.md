---
layout: post
title:  "Can Machine Learning be Lean?"
date:   2021-06-12
---

Lean was a way of improving manufacturing efficiency in [Toyota][toyota]. [Lean software][lean-software] development and [lean startup][lean-startup] methodologies followed it. One of the key take-aways of lean is cutting off the unnecessary processes while leaving the ones that bring actual value. Could the ideas be translated to machine learning and data science? Let's walk through the [seven][wastes-1] [deadly wastes][wastes-2] and see if they apply to machine learning.

## 1. Waste of defects

You could avoid many defects by testing the code. [Jeremy Jordan][jj-testing-ml], [Luigi Patruno][mlp-testing-ml], and [Martin Fowler's blog][cd4ml] make good points about testing machine learning-based software. Start with writing unit tests for the code and verifying the test error metrics. The less obvious ideas are smoke tests (running the whole pipeline to see if nothing "smokes") or having test data cases for which the model [needs to make][karpathy] correct predictions, etc.

Evaluating the [model fairness][fairness] is also valuable. Model making unfair (e.g. racist) predictions could induce reputational costs.

Lean manufacturing also introduced the idea of [andon][andon], instantly stopping the production line in case of a defect and prioritizing fixing it. We can apply it to machine learning as well.

Imagine you are building a linear regression model to predict the number of website visits. The model is [wrong, but it proved useful][models-wrong] because of being fast and easily interpretable. Before using it in production, you verified that negative predictions happen rarely. To prevent them completely, you wrote code replacing negative values with zeros. Now imagine a [data drift][data-drift] occurs and your algorithm starts returning a lot of zeroes. Debugging such issues, especially in complex systems, can be troublesome. Instead of lipsticking the pig, often it is wiser to [fail fast][fail-fast]. Maybe you shouldn't have replaced the values with zeros so the problems would be instantly visible? The less extreme solution is to monitor such cases and send alerts if their frequency increases.

## 2. Waste of inventory

In traditional software engineering partially done work is a common source of waste of inventory. The same applies to data science, but there are additional examples of waste specific to this field. Idle jobs, like virtual machines that were not closed, or unnecessarily repeated computations are waste. The less obvious ones may be using inadequate or costly technological solutions. Instead of [grid search][grid-search] for hyperparameter tuning, using the random search or Bayesian optimization might be more efficient. Using big data technologies (Spark) for small datasets is unnecessary at best (e.g. [Pafka Szilard][rf-benchmark] has shown that Spark's implementation of random forest can be less efficient than the regular implementations). Training a model not usable in a production environment (too slow, too high memory consumption) is also waste.

## 3. Waste of processing

The classic case of the waste of processing in software engineering is the unnecessary formal processes. For example, producing tons of drafts, documentation, reports, and PowerPoints that nobody reads.
[Doug Rose][agile-ds] has noticed that Scrum does not work well for data science teams. He is [not alone][ds-scrum] in this opinion. Data science tasks are hard to plan, the deliverables are less specific, they often force follow-ups that change the scope of the sprint, etc. In such a case, using Scrum for a data science team may lead to unnecessary processes implied by the framework.

## 4. Waste of waiting

In a lean production line, inventory flows smoothly between different workstations. Each workstation has a single responsibility with the workload balanced between the workstations to avoid downtime. While it's not the same, it's a good practice to run the machine learning tasks in modular pipelines (download the data, clean it, filter, split to train and test sets, engineer features, train, evaluate, publish, etc). It would not make anything faster but is easily extensible, modifiable, and debuggable.

Waiting for the model to finish training is the biggest waste of waiting. Unfortunately, it is also one of the hardest to avoid. To speed it up, you could use a more powerful machine. Such machines are more expensive, but consider the costs in the context of the hourly wage for the idle data scientists waiting for the results. Using [early stopping][early-stopping] of the training may shorten the training time and improve the quality of the results.

Waste of waiting may also be related to the popularity of frameworks such as PyTorch relatively to TensorFlow 1.x. Before TensorFlow introduced the [eager mode][eager-mode], users of PyTorch valued it because it made the work more interactive, giving instant feedback about the code.

After training a model, we usually wait for feedback from the users. Release the product early and often to get the feedback faster, as [Emmanuel Ameisen][ml-powered] suggests. [Extreme programming recommends][extreme-programming] even having the customers on-site. 

## 5. Waste of motion

Waste of motion is about unnecessary movements. When starting a data science project you need to meet the stakeholders, the potential customers, or domain experts to learn more about the problem, and the data owners to learn how to access the data, etc. Improving processes related to those tasks can reduce the waste of motion.
Using standardized templates, tools, APIs, code formatting (e.g. auto-formatting using Black), etc reduces unnecessary "movement" related to deciding on them on a case-by-case basis. Onboarding new employees or taking over someone's work is easier when projects are standardized. That's one of the reasons [Google][google] heavily uses standardization.

Automating the data and machine learning pipelines also reduces the waste of motion. Bash scripts, Airflow, or Luigi pipelines, can take care of the moving parts of the process. Version control keeps the scripts and notebooks in a single place, so there's no ambiguity about where to find them.

## 6. Waste of transportation

In software engineering, [task switching][multitasking] is considered a waste of transportation. Focusing on one thing at a time, even if it causes some [slack time][slack-time], makes you more, not less efficient. In data science, moving the data between our workers and databases also falls into this category. Using feature stores for the clean, pre-computed features is an example of reducing waste.

## 7. Waste of overproduction

Adding unnecessary features to the software is a waste of overproduction. Machine learning projects tend to take much longer than planned. It is always possible to tune the hyperparameters more, try different models, clean the data better, etc to improve the prediction accuracy. Those gains are not always worth it. Starting with a [simple model][simple-model] (rule-based, logistic regression, decision tree) may be a good start. The simple model may turn out good enough otherwise it will become a benchmark for a more complicated one. As described in [*Building Machine Learning Powered Applications*][ml-powered], starting with a simple model is a chance to build the supporting infrastructure ahead. The simple model serves as a minimum viable product to get feedback from the potential users early on.

## Are we there yet?

Software engineering has built many tools to become more agile and lean, data science and machine learning are a bit behind. [MLOps][mlops-book] tries bringing the DevOps ideas into the data science ground. We are currently observing the emergence of different tools and ideas for improving the processes and productionalizing machine learning models. Applying lean thinking principles to machine learning projects can lead to better utilization of resources.


 [agile-ds]: https://www.linkedin.com/learning/learning-data-science-using-agile-methodology/welcome
 [andon]: https://en.wikipedia.org/wiki/Andon_(manufacturing)
 [cd4ml]: https://martinfowler.com/articles/cd4ml.htmls
 [data-drift]: https://blog.dataiku.com/a-primer-on-data-drift
 [ds-scrum]: https://www.datascience-pm.com/scrum/
 [eager-mode]: https://ai.googleblog.com/2017/10/eager-execution-imperative-define-by.html
 [early-stopping]: https://citeseer.ist.psu.edu/viewdoc/summary?doi=10.1.1.37.900
 [extreme-programming]: https://www.agilealliance.org/glossary/xp/
 [fail-fast]: https://www.ibm.com/garage/method/practices/culture/failing-fast
 [fairness]: https://fairmlbook.org/
 [google]: https://www.goodreads.com/book/show/48816586-software-engineering-at-google
 [grid-search]: https://stats.stackexchange.com/questions/160479/practical-hyperparameter-optimization-random-vs-grid-search
 [jj-testing-ml]: https://www.jeremyjordan.me/testing-ml/
 [karpathy]: https://www.youtube.com/watch?v=hx7BXih7zx8
 [lean-software]: https://www.goodreads.com/book/show/194338.Lean_Software_Development
 [lean-startup]: https://www.goodreads.com/book/show/10127019-the-lean-startup
 [ml-powered]: https://www.goodreads.com/book/show/50204636-building-machine-learning-powered-applications
 [mlops-book]: https://pages.dataiku.com/oreilly-introducing-mlops
 [mlp-testing-ml]: https://mlinproduction.com/testing-machine-learning-models-deployment-series-07/
 [models-wrong]: https://stats.stackexchange.com/questions/57407/what-is-the-meaning-of-all-models-are-wrong-but-some-are-useful
 [multitasking]: https://www.apa.org/research/action/multitask
 [rf-benchmark]: https://github.com/szilard/benchm-ml
 [simple-model]: https://mlpowered.com/posts/start-with-a-stupid-model/
 [slack-time]: https://twitter.com/allenholub/status/1403063770503548930
 [toyota]: https://en.wikipedia.org/wiki/Toyota_Production_System
 [wastes-1]: https://www.spica.com/blog/7-wastes-of-lean
 [wastes-2]: https://www.linkedin.com/learning/dealing-with-the-seven-deadly-wastes/seven-wastes-overview