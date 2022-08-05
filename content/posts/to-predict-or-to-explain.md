---
title: "To Predict or to Explain"
date: 2017-05-24T10:10:30+03:00
tags: ["machine learning", "explanatory"]
---

As data scientists, our day job is around modelling. We create models to recommend new products, to increase conversion rates, to explain user behaviour etc. And depending on your background, it is more likely to be familiar either with machine learning techniques or regression type analysis. It is the difference among these two distinct approaches in modelling that motivated me to write this post which is a summary of a talk I gave recently in PyData London.

Generally speaking, we model so as to achieve one of two goals a) *explain* what is going on in our observations, or b) *predict* given an unseen observation. In a perfect, noise free world, these two endeavours would converge to the same model, which would be the model that produces the data we observe but in reality there are different trade-offs to be made depending on the goal of your model.

These differences arise from two main areas:
- the *operationalisation* of theoretical *constructs*
- the *bias-variance* trade-off

Theoretical constructs, such as intelligence, give us the ability to explain and articulate our causal hypothesis. At the same time, in order to create a model, we need operationalised versions of these constructs which are measurable, such as I.Q. score, in order to do modelling. It is that distance between the theoretical construct and its equivalent measurable definition that justifies part of the difference between the two approaches.

At the same time when doing modelling, we seek to minimise the expected error. This can be decomposed in two terms, the Bias and the Variance. Bias represents the distance between the actual model producing the data and the one we use while variance captures the complexity of the model. When our goal is to do statistical modelling to test a causal hypothesis, i.e. explain, we focus on minimising the distance between the actual model and the one we constructed, thus minimise bias. On the other hand when predicting we care mostly about minimising the combined bias and variance, and sometimes this means sacrificing model accuracy, thus increasing bias, for minimising variance, an example of which is regularisation.

&nbsp;
![bias-variance](/images/bias-variance.png#center)
&nbsp;

While these are the high level reasons why statistical modelling for prediction and explanation are different, these differences manifest in different ways in all steps of creating a model. The process of building a model in machine learning looks like this:

&nbsp;
![ml-process](/images/ml-process.png#center)
&nbsp;

Starting from the *pre processing* step, the volume of data used tends to be dramatically different. On the one hand, explanatory modelling utilises as little data as possible in order to produce statistically significant results and thus one of the first steps usually is to calculate the sample size needed for that. Predictive modelling, on the other hand, is significantly more data hungry, since every additional data point can have a significant impact on the ability of the model to generalise which translates into increased predictive accuracy.

*Feature engineering* surfaces two more differences. Features (operationalised constructs) in explanatory modelling have a very specific role, and that is to test an underlying theory or hypothesis. This means that any transformation of the variables that makes them uninterpretable, such as PCA, or addition of unrelated to the theory variables is avoided since it does not increase our ability to explain even if it increases our ability to predict. The other difference has to do with the availability of a feature at prediction time. When modelling for prediction, it only makes sense to use features that are available at the time we wish to make the prediction, a restriction that does not apply when our goal is to best explain our observable world.

During the *modelling* step, the choice of model is governed heavily from the underlying goal of modelling. This is because different models score differently in their ability to explain versus their ability to predict. On the one end of the spectrum, neural networks are powerful models with the ability to model complex relationships from the dataset but which offer us with little intuition on how they work, and on the other end, linear regression is a very simple model that rarely performs well in prediction but gives us a very clear view on the contribution of the different features.

&nbsp;
![predict-explain](/images/predict-explain.png)
&nbsp;

The final difference comes into play during the *model evaluation* step. It should come as not surprise at this point that there are different ways to evaluate performance in each case. Explanatory modelling asks for a way to assess explanatory power and finds it usually in the form of R² which quantifies the amount of variance explained by our model. Predictive modelling cares only for predictive power and thus quantifies success by using predictive accuracy.

At this point, I hope that I have convinced you that modelling for prediction is not the same as modelling for explanation which leads me to my two take away points:
- predictive modelling techniques perform better at prediction tasks
- there are scientific and ethical reasons to explain our black box techniques

The first point might sound obvious but there are a number of cases where explanatory techniques are being used to make predictions, for example election forecasting, something that often results in poor results.

The second point is a bit more nuanced but what I mean is that if we want to better expand our understanding of how nature works, it will not be enough to create models that are good in replicating nature. Neural networks offer a perfect example for this, because even though they have been proven to work better than the human brain in quite complex tasks, they have not assisted much in understanding how our brain works. The ethical dimension comes from the fact, that more and more lately, we are using algorithms to replace human decision making, and even though humans come with their own biases, this is no excuse for replicating these biases into the algorithms we develop. We do not want models that are neither racial nor sexually biased to make hiring or legal decisions whereas at the moment we have both.

I leave you with a final thought, next time you develop a model why not ask first, what is the goal of this model? To predict or to explain? And then use the most relevant technique for the task.

For more details you can:
- [watch my PyData talk](https://www.youtube.com/watch?v=3ywnb3W-hNU&t=2s)
- [read the paper that inspired me to give the talk](https://arxiv.org/pdf/1101.0891.pdf)
- [watch the presentation of the author of the paper](https://www.youtube.com/watch?v=vWH_HNfQVRI)