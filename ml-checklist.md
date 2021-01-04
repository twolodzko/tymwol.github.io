# Deploying Machine Learning Models Checklist

In [The Checklist Manifesto][checklist] Atul Gawande shows how using checklists can make
everyone's work more efficient and less error prone. If checklists are useful for
aircraft pilots and surgeons, they should be useful for deploying machine learning models
as well. By no means the checklist would be exhaustive, it would be an impossible task.
I'm writing it mostly for myself, to write down the things that even if sound obvious,
can easily be forgotten, or skipped. You'll notice some repetitions, this is by design.

This checklist is based on the [Introducing MLOps][mlops-book] book by Mark Treveil et al,
the [Building Machine Learning Powered Applications][ml-powered] book by Emmanuel Ameisen,
the free [Full Stack Deep Learning][fsdl] course, the [Rules of Machine Learning][ml-rules]
document by Martin Zinkevich, and [ML Ops: Operationalizing Data Science][mlops-report]
report by David Sweenor et al, some other resources, and my own experience.

## 1. What problem are you trying to solve?

*At this stage we are **not** thinking in terms of machine learning metrics, but business objectives.
Beware of the [Goodhart's law](https://en.wikipedia.org/wiki/Goodhart%27s_law), your aim is to solve
some problem, not to play the metric at all cost.*

 * In plain English, what are you trying to do?
 * Should you do it?

   *Some problems are ill-posed, they sound reasonable at first, but in fact the actual underlying
   problem is different (e.g. [X-Y problem][xy]). Another scenario where you shouldn't do it,
   is when the technology could have harmful side-effects (e.g. [résumé screening][amz-resume-screen]
   algorithm trained on historical data could amplify the social biases in the recruitment process).*

 * Are there clear [performance indicators][first-objective] that would enable you to
 measure success?
 * What is the [definition of done][dod]? Can we define an [acceptance test][acceptance-test]
 for this problem?
 * Do you actually need machine learning for that? Are the costs of using machine learning worth it?
 * Do you have enough resources to solve it (time, knowledge, people, compute)?

## 2. Do you have the data?

 * Do you have access to the data that is needed for solving this problem?
 If not, do you have way of gathering it?
 * Can this data be used (terms of use, privacy, etc)? Does it contain any entries that would
 need to be removed, or anonymized?
 * Is the data labelled? Do you have an efficient way of labelling it?

   *As noticed by [Richard Socher][socher-tweet], "Rather than spending a month figuring out an
   unsupervised machine learning problem, just label some data for a week and train a classifier".
   [Data labeling][labeling] may not be the sexiest part, but it's a time well invested.*

 * Would this data be available at the time the model is deployed into production? Would you be
 using same, or different, source of data?
 * Is this data representative for the population of interest?

   *While the more data we have, the better, it is not only about quantity,
   but also quality. As discussed by Xiao-Li Meng in the [Statistical paradises and paradoxes in Big Data][big-data-paradoxes]
   talk, having a lot of bad data does not make us any closer to the solution.*

 * Can using this data lead to obtaining biased results? Are minority populations sufficiently
 well represented?

## 3. Do you have a baseline?

*[Lean Startup][lean] has introduced the idea of [minimum viable product (MVP)][mvp], the simplest solution that
"does the job". Before building a full-blown machine learning model, first try the cheap and easy solution like
rule-based system, decision tree, linear regression etc. This would help you with framing the problem, but also
can be used to gather initial feedback ("is this what you need?"), and would serve a baseline, so that you can
benchmark your model against it. Emmanuel Ameisen makes similar points in [his book][ml-powered], and
[this blog post][solve-nlp], there's also a nice [talk about baselines][baselines].*

 * What is your baseline? How was the problem solved before (not necessary using machine learning)?
 Do you have access to metrics, that would enable you to compare current solution with the machine learning one?
 * Was this, or similar, problem solved using machine learning by anyone before (literature)? What was the performance
 of those solutions?

## 4. Is the model ready for deployment?

*At this stage the data science magic happens. Data scientists conduct exploratory data analysis, clean the data,
preprocess it, conduct feature engineering (see [Zheng & Casari][feature-engineering]), and validate the model.
While all those steps are already covered in 1001 online tutorials and books, there are things that can be easily
overlooked while being important from the deployment point of view.*

### 4.1. Are the data preprocessing steps documented?

 * Did you conduct the exploratory data analysis? What are the potential problems with using this data?
 * Are the assumptions you make about the data documented? Can they be transformed into code (e.g. it is assumed
 that some variable should fit some bounds, so we can use assertion that checks if the values actually fit it)?
 * Was the data cleaning, preprocessing, and feature engineering documented?
 Can those processes be replicated in the production environment?

### 4.2. Does it work?

*See the [Evaluating Machine Learning Models][evaluating-ml] book by Alice Zheng.*

 * Does the code run  (e.g. the Jupyter notebook does not crash)?
 * Was it proven that the model actually solves the problem you were trying to solve?
 * What metrics should be used to assess the model performance? Is the performance acceptable?
 * Did you check for overfitting?
 * Are there any potential [data leaks][data-leak] that might have inflated the performance?
 * Are the results reproducible? Is it documented (as a code) how to reproduce them?

### 4.3. Did you explore the predictions made by the model?

*In some industries (e.g. finance) being able to explain the predictions is required by law. In many other
areas model explainability and fairness may be important as well, or at least useful as sanity checks.
For more details check the [Interpretable Machine Learning][interpretable-ml] book by Christoph Molnar.*

 * Do the predictions resemble the real data (so-called [posterior predictive checks][ppc])? Are they realistic?
 * Are you able to explain the predictions ([partial dependence plots][pdp], subpopulation analysis, [Shapley values][shapley],
 what-if analysis)?
 * Did you check for biasses (e.g. gender, race)?
 * Did you manually check some of the misclassified examples? When does the model make mistakes?

### 4.4. Does the code meet the quality standards?

 * Is the model code documented well enough, so that other people can work with it?
 * Are the dependencies of the code documented (preferably something like Docker image, virtual enviroment,
 or at worst, a list of dependencies and the package versions)?
 * Does the model fit in the technical constraints (technology used, memory consumption, training time,
 prediction time etc)?

   *Production enviroment might be using a different setup, or even the model might get re-implemented,
   but it might be useful to ask this question early on, to avoid surprises.*

### 4.5. Do you have tests for the model?

*[Jeremy Jordan][testing-ml] makes a good distinction between unit tests for the code and model tests.
Additionally, the [Explore It!][explore-it] book by Elisabeth Hendrickson is a nice introduction to
exploratory testing, that could serve as an inspiration on how to test the black-box'ish machine learning
code.*

 * Is the model code accompanied by the unit tests? Is the test coverage acceptable?
 * Is there a documented way to conduct a smoke test of the code, to make sure it doesn't fail?
 * Do you have functional tests proving that the model works reasonably, for a reasonably realistic data?
 * Do you have tests checking how does the model behave for extreme cases (e.g. zeroes, very low, or very high
 values, missing data, noise, adverserial examples)?

## 5. Do you know everything to be able to safely deploy it?

 * Do you have sufficient infrastructure and other resources (e.g. Ops engineers) to deploy it?
 * How the model is going to be deployed (e.g. microservice, package)?
 How would it be used?
 * Is it going to run real-time, or in batch mode?
 * What computational resources are needed (e.g. GPUs, memory)?
 * How does it interact with other services, or parts of the software? What could go wrong?
 * Do you know the requirements needed by it (e.g. package versions)?
 * Do you need to make any extra steps if the model makes anomalous predictions (e.g. if
 predictions pass some threshold, fall-back to rule-based system)?
 * What artifacts (e.g. model parameters) need to be saved? How and where would you store them?
 * How would you handle model versioning and data versioning?
 * What tests you will run for the code? How often?
 * How would you deploy new version of the model (canary deployment, A/B testing)?
 * How often do you need to re-train the model? What is the upper bound ("at least") and
 lower bound ("not sooner than")?

## 6. How would you monitor the model?

 * How would you gather the "hard" metrics (runtime, memory consumption, compute, disk space)?
 * What [KPI's][kpi] need to be monitored?
 * What data quality and [model metrics][evaluating-ml] you need to monitor in production?
 * When replacing a model with a new one, what metrics would be needed to make the decision?
 * How would you monitor model degradation?

   a. ***Ground truth evaluation**, where the predictions are compared to the labeled data,
      so that the drop in performance (model metrics, business metrics) would be observed
      in case of drift.*  
   b. ***Input Drift Detection** means monitoring the distribution of the data over time.
      This can be achieved by:*  
      - *Monitoring the summary statistics  (e.g. mean, standard deviation, minimum, maximum),
      or using formal tests ([K-S tests][ks], [chi-squared tests][chisq]) to detect anomalies
      in the input data.*  
      - *Compare the distributions of predictions made by the model on old vs new data (K-S test).*  
      - *Using domain classifier, i.e. a classification algorithm that tries to predict
      old vs new data, and if it is successful, it suggests that the data might have
      changed.*

   *For more details see Chapter 7 from the [Introducing MLOps][mlops-book] book and the
   [A Primer on Data Drift & Drift Detection Techniques][drift-primer] whitepaper by Dataiku.*

 * Are there any potential feedback loops that need special monitoring?
 
   *For example, if you are recommending videos to the users based on their viewing history,
   users would be more likely watch the videos you are serving them as recommendations.
   As a consequence, your future data would be influenced by the recommendation algorithm,
   so if you re-trained the algorithm on such data, you would be amplifying the recommendations
   you already made.*

 * How would you be able to [access the metrics][metrics-jordan] (e.g. Kibana, Grafana, MLflow, Neptune)?
 * Who's going to monitor the metrics?

## 7. Can it be used?

*At least some of those considerations should, and will, be made before starting the project, but before
deployment you should ask the questions one more time.*

 * Do the benefits of using the model outweigh the cost of developing, deploying, and maintaining it?
 * What are the risks related to using the model (financial, business, legal, safety, reputational)?
 * Does using it comply with the regulations (e.g. [GDPR][gdpr])?
 * Did you make sure the model is not biased (e.g. race, gender) or does not have any harmful side-effects?
 * If the model predictions would need to be audited (legal obligations), are you storing
 all the necessary artifacts (model code, parameters, data used for training)?



 [checklist]: https://www.goodreads.com/book/show/6667514-the-checklist-manifesto
 [mlops-book]: https://pages.dataiku.com/oreilly-introducing-mlops
 [ml-powered]: https://www.goodreads.com/book/show/50204636-building-machine-learning-powered-applications
 [fsdl]: https://course.fullstackdeeplearning.com/
 [ml-rules]: https://developers.google.com/machine-learning/guides/rules-of-ml
 [mlops-report]: https://www.tibco.com/resources/ebook-online/ml-ops-operationalizing-data-science-four-step-approach-realizing-value-data
 [baselines]: https://course.fullstackdeeplearning.com/course-content/setting-up-machine-learning-projects/baselines
 [lean]: https://www.goodreads.com/book/show/10127019-the-lean-startup
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
 [pdp]: https://christophm.github.io/interpretable-ml-book/pdp.html
 [shapley]: https://christophm.github.io/interpretable-ml-book/shapley.html
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
