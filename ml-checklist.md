---
layout:        post
title:         "Deploying Machine Learning Models: A Checklist"
date:          2021-01-05
modified_date: 2021-01-08 
---

In [The Checklist Manifesto][checklist], Atul Gawande shows how using checklists can make everyone's
work more efficient and less error-prone. If they are useful for aircraft pilots and surgeons, we could
use them to help us with deploying machine learning models as well. While most of those steps might sound
obvious, it's easy to forget them, or leave them for "somebody" to do "later". In many cases, skipping those
steps will sooner or later lead to problems, hence it's good to have them as a checklist.

For more details, you can check resources like [Introducing MLOps][mlops-book]
book by Mark Treveil et al, [Building Machine Learning Powered Applications][ml-powered]
book by Emmanuel Ameisen, the free [Full Stack Deep Learning][fsdl] course,
[Rules of Machine Learning][ml-rules] document by Martin Zinkevich,
[ML Ops: Operationalizing Data Science][mlops-report] report by David Sweenor et al,
the [Responsible Machine Learning][responsible-ml-report] report by Patrick Hall et al,
the [Continuous Delivery for Machine Learning][cd-for-ml] article by Danilo Sato et al,
[Machine Learning Systems Design][ml-systems-design] page by Chip Huyen, and the
[ml-ops.org](https://ml-ops.org/) webpage.

## 1. What problem are you trying to solve?

*At this stage, we are focusing on business objectives. Beware of [Goodhart's law][goodhart-law],
you aim to solve some problem, rather than to play the metric at all costs.*

 * In plain English, what are you trying to do?
 * Should you do it?

   *Some problems are ill-posed, they may sound reasonable at first, but the actual underlying
   problem is different (e.g. [X-Y problem][xy]). Another scenario where you shouldn't do it
   is when the technology could have harmful side-effects (e.g. [résumé screening][amz-resume-screen]
   algorithm trained on historical data could amplify the social biases in the recruitment process).*

 * What is the [definition of done][dod]? Can we define [acceptance tests][acceptance-test]?
 * Are there clear [performance indicators][first-objective] that would enable you to
 measure success?
 * Do you need machine learning for that? Are the costs of using machine learning worth it?
 * Do you have enough resources to solve it (time, knowledge, people, compute)?

## 2. Do you have the data?

 * Do you have access to all the data that is needed for solving the problem?
 If not, do you have a way of gathering it?
 * Would this data be available in the production environment?
 * Can this data be used (terms of use, privacy, etc)? Does it contain any sensitive information
 that cannot be used, or needs to be anonymized?
 * Is the data labeled? Do you have an efficient way of labeling it?

   *As tweeted by [Richard Socher][socher-tweet]: "Rather than spending a month figuring out an
   unsupervised machine learning problem, just label some data for a week and train a classifier".
   [Data labeling][labeling] may not be the sexiest part of the job, but it's a time well invested.*

 * Is the data up-to-date and accurate? Did you check how accurate the labels are?
 * Is this data representative of the population of interest? What is your [population][population]?

   *While the more data we have, the better, it is not only about quantity,
   but also quality. As discussed by Xiao-Li Meng in the [Statistical paradises and paradoxes in Big Data][big-data-paradoxes]
   talk, having a lot of bad data does not make us any closer to the solution.*

 * Could using this data lead to obtaining biased results? Are the minority groups sufficiently
 well represented?

## 3. Do you have a baseline?

*[Lean Startup][lean] has introduced the idea of the [minimum viable product (MVP)][mvp], the simplest solution that
"does the job". Before building a full-blown machine learning model, first try the cheap and easy solution like
rule-based system, decision tree, linear regression, etc. This would help with framing the problem, can be [used
to gather initial feedback][ml-product] ("is this what you need?"), and would serve as a baseline. Emmanuel Ameisen
makes similar points in [his book][ml-powered], and in [this blog post][solve-nlp], there's also a nice
[talk about the baselines][baselines].*

 * What is your baseline? How was the problem solved before (not necessarily using machine learning)?
 * Do you have access to the metrics needed to compare your solution with the baseline?
 * Has anyone used machine learning to solve similar problems before (literature)? What did we learn
 from that?

## 4. Is the model ready for deployment?

*At this stage, data science magic happens. Data scientists conduct exploratory data analysis, clean the data,
preprocess it, conduct feature engineering (see [Zheng & Casari][feature-engineering]), train, tune, and validate
the model.*

### 4.1. Are the data preprocessing steps documented?

 * Did you conduct the exploratory data analysis? What are the potential problems with this data?
 * Are the assumptions made about the data documented? Can they be transformed into automated data checks?
 * Are the data cleaning, preprocessing, and feature engineering steps documented? Would it be possible to
 replicate them in the production environment?
 * How would you handle missing data in production?

### 4.2. Does it work?

*See the [Evaluating Machine Learning Models][evaluating-ml] book by Alice Zheng.*

 * Does the code run  (e.g. the Jupyter notebook does not crash)?
 * Was it proven that the model solves the problem you were trying to solve?
 * What metrics should be used to assess the performance of the model? Is the performance acceptable?
 * Did you check for overfitting?
 * Could any [data leaks][data-leak] have inflated the performance?
 * Is it documented (as a code) how to reproduce the results? Are they reproducible?

### 4.3. Did you explore the predictions?

*In some industries being able to explain the predictions is required by law. In many other cases,
model explainability and fairness may be equally important, or at least useful as sanity checks.
For more details, check the [Interpretable Machine Learning][interpretable-ml] book by Christoph Molnar
and the [Real-World Strategies for Model Debugging][model-debugging] post by Patrick Hall.*

 * Are the predictions reasonable? Do they [resemble the real data][ppc]?
 * Would you be able to explain the predictions to your grandmother (partial dependence plots, subpopulation analysis,
 Shapley values, LIME, what-if analysis, residual analysis)?
 * Did you check for biases (e.g. gender, race)?
 * Did you manually check some of the misclassified examples? When does the model make mistakes?

### 4.4. Does the code meet the quality standards?

 * Is the code documented well enough, so that other people would be able to use it?
 * Are the dependencies documented (Docker image, virtual environment, a list of all the packages and their versions)?
 * Does it meet the technical constraints (technology used, memory consumption, training time,
 prediction time, etc)?

### 4.5. Do you have the tests for the model?

*[Jeremy Jordan][testing-ml] makes a good distinction between unit tests for the code, and model tests.
Additionally, the [Explore It!][explore-it] book on exploratory testing by Elisabeth Hendrickson, may serve
as an inspiration on how to test the black-box'ish machine learning code.*

 * Is the model code accompanied by the unit tests? Is the test coverage acceptable?
 * Is there a documented way to run a smoke test?
 * Do you have functional tests proving that the model works reasonably, for reasonably realistic data?
 * Do you have tests checking how it behaves for extreme cases (e.g. zeroes, very low, or very high
 values, missing data, noise, adversarial examples)?

## 5. Do you know everything needed to deploy it?

 * Do you have sufficient resources to deploy it (e.g. infrastructure, the help of DevOps engineers)?
 * How is it going to be deployed (e.g. microservice, package, stand-alone app)?
 * Will it run in real-time, or in batch mode?
 * What computational resources are needed (e.g. GPUs, memory)?
 * How does it interact with other services or parts of the software? What could go wrong?
 * Do you know all it's dependencies (package versions)?
 * Do you need to make any extra steps if the model makes anomalous predictions (e.g. truncate them,
 or if predictions pass some threshold, fall-back to the rule-based system)?
 * What metadata and artifacts (e.g. model parameters) need to be saved? How are you going to store them?
 * How would you handle model versioning and data versioning?
 * What tests will you run for the code? How often?
 * How would you deploy a new version of the model (manual inspection, canary deployment, A/B testing)?
 * How often do you need to re-train the model? What is the upper bound ("at least") and
 lower bound ("not sooner than")?

## 6. How would you monitor it?

 * How would you gather the "hard" metrics (runtime, memory consumption, compute, disk space)?
 * What data quality and [model metrics][evaluating-ml] you need to monitor in the production?
 * What [KPI's][kpi] need to be monitored?
 * When deploying a new model, what metrics would be needed to decide between switching
 between the models?
 * How would you monitor input drift and model degradation?

   a. ***Ground truth evaluation**, where the predictions are compared to the labeled data
      so that the drop in performance (model metrics, business metrics) would be observed
      in case of drift.*  
   b. ***Input Drift Detection** means monitoring the distribution of the data over time.
      This can be achieved by:*  
      - *Monitoring the summary statistics  (e.g. mean, standard deviation, minimum, maximum),
      or using formal tests ([K-S tests][ks], [chi-squared tests][chisq]) to detect anomalies
      in the input data.*  
      - *Compare the distributions of predictions made by the model on old vs new data (K-S test).*  
      - *Using a domain classifier, i.e. a classification algorithm that tries to predict
      old vs new data, and if it is successful, it suggests that the data might have
      changed.*  
   
   *For more details see Chapter 7 from the [Introducing MLOps][mlops-book] book and the
   [A Primer on Data Drift & Drift Detection Techniques][drift-primer] whitepaper by Dataiku.*

 * Are there any potential feedback loops that need special attention?
 
   *For example, if you are recommending videos to the users based on their viewing history,
   users would be more likely to watch the videos you are serving them as recommendations.
   As a consequence, your future data would be influenced by the recommendation algorithm,
   so if you re-trained the algorithm on such data, you would be amplifying the recommendations
   you already made.*

 * Would you be able to easily [access the metrics][metrics-jordan] (e.g. Kibana, Grafana, MLflow, Neptune)?
 * Who's going to monitor the metrics?

## 7. Can it be used?

*At least some of those considerations should, and would, be made before starting the project, but before
deployment, you should ask the questions one more time. For more details see the 
[Responsible Machine Learning][responsible-ml-report] report, and the [Responsible AI Practices][responsible-ai-practices]
webpage by Google.*

 * Was it tested in a production-like environment to make sure it works the same as during development?
 * Was there an external review of the results (e.g. domain experts)?
 * Do the benefits of using the model outweigh the cost of developing, deploying, and maintaining it?
 * Was it reviewed in terms of fairness (e.g. race, gender)?
 * Have you considered what are the potential misuses, or harmful side-effects of using the model?
 * Does using it comply with the regulations (e.g. [GDPR][gdpr])?
 * If the predictions would need to be audited (legal obligations), are you storing all the necessary
 artifacts (version-controlled code, parameters, data used for training)?
 * Is there a fall-back strategy in case it breaks, or other problems with the algorithm?



 [checklist]: https://www.goodreads.com/book/show/6667514-the-checklist-manifesto
 [mlops-book]: https://pages.dataiku.com/oreilly-introducing-mlops
 [ml-powered]: https://www.goodreads.com/book/show/50204636-building-machine-learning-powered-applications
 [fsdl]: https://course.fullstackdeeplearning.com/
 [ml-rules]: https://developers.google.com/machine-learning/guides/rules-of-ml
 [mlops-report]: https://www.tibco.com/resources/ebook-online/ml-ops-operationalizing-data-science-four-step-approach-realizing-value-data
 [responsible-ml-report]: https://www.h2o.ai/resources/ebook/responsible-machine-learning/
 [cd-for-ml]: https://martinfowler.com/articles/cd4ml.html
 [baselines]: https://course.fullstackdeeplearning.com/course-content/setting-up-machine-learning-projects/baselines
 [lean]: https://www.youtube.com/watch?v=fEvKo90qBns
 [mvp]: https://www.agilealliance.org/glossary/mvp/
 [solve-nlp]: https://blog.insightdatascience.com/how-to-solve-90-of-nlp-problems-a-step-by-step-guide-fda605278e4e
 [feature-engineering]: https://www.goodreads.com/book/show/31393737-feature-engineering-for-machine-learning
 [big-data-paradoxes]: https://www.youtube.com/watch?v=8YLdIDOMEZs
 [testing-ml]: https://www.jeremyjordan.me/testing-ml/
 [explore-it]: https://www.goodreads.com/book/show/15980494-explore-it
 [ppc]: https://stats.stackexchange.com/a/125576/35989
 [socher-tweet]: https://twitter.com/RichardSocher/status/840333380130553856
 [labeling]: https://course.fullstackdeeplearning.com/course-content/data-management/labeling
 [interpretable-ml]: https://christophm.github.io/interpretable-ml-book/
 [model-debugging]: https://towardsdatascience.com/strategies-for-model-debugging-aa822f1097ce
 [evaluating-ml]: https://edu.heibai.org/evaluating-machine-learning-models.pdf
 [ks]: https://en.wikipedia.org/wiki/Kolmogorov%E2%80%93Smirnov_test
 [chisq]: https://en.wikipedia.org/wiki/Chi-squared_test
 [kpi]: https://kpi.org/KPI-Basics
 [gdpr]: https://gdpr.eu/what-is-gdpr/
 [acceptance-test]: https://www.agilealliance.org/glossary/acceptance/
 [dod]: https://www.agilealliance.org/glossary/definition-of-done/
 [first-objective]: https://developers.google.com/machine-learning/guides/rules-of-ml#your_first_objective
 [data-leak]: https://www.kaggle.com/dansbecker/data-leakage
 [metrics-jordan]: https://www.jeremyjordan.me/ml-monitoring/
 [drift-primer]: https://pages.dataiku.com/data-drift-detection-techniques
 [xy]: https://meta.stackexchange.com/questions/66377/what-is-the-xy-problem
 [amz-resume-screen]: https://www.reuters.com/article/us-amazon-com-jobs-automation-insight-idUSKCN1MK08G
 [responsible-ai-practices]: https://ai.google/responsibilities/responsible-ai-practices/
 [goodhart-law]: https://www.fast.ai/2019/09/24/metrics/
 [population]: https://pubmed.ncbi.nlm.nih.gov/23216426/
 [ml-product]: https://www.jeremyjordan.me/ml-requirements/
 [ml-systems-design]: https://huyenchip.com/machine-learning-systems-design/toc.html
