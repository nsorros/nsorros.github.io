---
title: "Assesing the fairness of our machine learning pipeline"
date: 2020-03-10T10:10:30+03:00
tags: ["machine learning", "fairness"]
---

My day to day job is to develop technologies that automate different processes at Wellcome through data science and machine learning. As a builder of these digital products, my team and I have to consider the unintended consequences they may have on users and on society more broadly. We know this is important because of famous cases where the consequences weren’t considered and harm has been done. For example the [Google photos tagging system](https://www.theverge.com/2018/1/12/16882408/google-racist-gorillas-photo-recognition-algorithm-ai) and [Amazon hiring algorithm](https://www.reuters.com/article/us-amazon-com-jobs-automation-insight/amazon-scraps-secret-ai-recruiting-tool-that-showed-bias-against-women-idUSKCN1MK08G) that have been accused of unintended racist and sexist bias. At Wellcome Data Labs we are developing an agile way of assessing the ethics and fairness behind the tools that we build. We hope that this will help us to anticipate the potential harm of our products so we can avoid causing it in the first place

One particular technology that has been in the spotlight lately is machine learning. This is for two main reasons. Firstly, machine learning can replicate and augment human biases that may be hidden in the data that the algorithm is trained on. Secondly the decisions made by machine learning algorithms are often black boxes — meaning that it is hard to decipher how the decision is made. This is particularly problematic when the decision that is being automated using machine learning has to do with a human, such as whether someone should get a loan or be released early from prison. At Wellcome Data Labs we work in a less risky environment and none of the decisions we automate are as life changing as these examples. We also use a form of machine learning (Natural Language Processing or NLP) where to best of our knowledge there have been fewer known incidents of bias.

## Our product — Reach

Our product [Reach](https://github.com/wellcometrust/reach) is a tool that helps researchers, policy makers and funders better understand how scientific research is used in policy. Reach consists of a pipeline of machine learning algorithms each of which could introduce bias into the product. As such we wanted to assess whether any of the algorithms contained any detectable bias and then take steps to mitigate it. Conceptually, what Reach does is very simple; the pipeline of algorithms finds reference sections in the text of policy documents and turns them into structured references. There is one algorithm for splitting references, one for extracting the individual parts of a reference -such as title, author, etc. — and one for matching references with a database of known publications.

&nbsp;
![reference-extraction-pipeline](/images/reference-extraction-pipeline.png#center)
&nbsp;

We decided to start our ethical evaluation on the algorithm that is responsible for extracting titles from the references. While we were aware that we could find bias in any part of the reference, i.e. the journal title, publication type, year, etc., we focussed on titles. This is because we use titles to cross-reference the output of the algorithmic pipeline.

## Defining fairness mathematically

To evaluate fairness we first need to define what we mean by ‘fairness’ and then find an appropriate metric to measure it in our product. There is no shortage of definitions of fairness, but we wanted to focus on group fairness which quantifies fairness in a group rather than at an individual level. For group fairness, there were a few definitions which we could have used.

An example would be demographic parity which would mean that we would need to extract an equal number of titles per group, for example per gender or journal. This would not make sense in our case since the distribution of authors or journals is not always 50/50. A particularly problematic definition is accuracy parity which asks if the algorithm makes the same number of mistakes per group. As long as the algorithm makes the same number of mistakes irrespective of the type, the algorithm shows accuracy parity and is ‘fair’. To understand why this is an issue, we first need to understand more about the types of mistakes an algorithm can make.

## Types of mistakes

There are two different types of mistakes that someone might look to equalise across groups: false positives and false negatives. In our example of an algorithm that decides if people should get a loan, false positives are when the algorithm wrongly gives someone a loan and false negatives are when the algorithm decides someone shouldn’t have a loan when they should. There are also two different types of correct predictions that we might want to equalise: true positives and true negatives. In the same example true positives is when the algorithm correctly predicts that someone should get a loan and true negatives when a loan is not being given to someone that would not repay it. It is important to note that it has been proven that optimising for all metrics is mathematically impossible except for some rare occasions. This means that the algorithm will always be ‘unfair’ by some definition, which is why it is so important to prioritise the type of error that would be most harmful in each specific context.

&nbsp;
![confusion-matrix](/images/confusion-matrix.png#center)
&nbsp;

## The right metric depends on the context

False positives might be more important in hiring- where the error of hiring the wrong person is difficult to correct- whereas false negatives are crucial in medical diagnosis. For Reach, we decided to prioritise true positive rate because we want to ensure that we extract titles equally accurately across all groups. We have decided this to avoid the hypothetical situation where we would be reinforcing a societal bias towards rewarding citations to a particular group more. Every metric is a trade off, and no one metric can capture everything. In the case of true positive rate, the metric misses out at over attribution. This is when parts of the reference which aren’t titles (e.g. names or dates) are being misclassified as titles and then spuriously matched with publications which generates false citations…

## The groups

The last piece missing before applying our definition of fairness to our context is a decision about the groups that are relevant. These groups people usually focus on when looking at fairness are often around protected characteristics such as gender and race. Like the definition of fairness, the specific groups to focus on also depend on the context of the decision that is being made by the algorithm. In the case of scientific publications, we decided to start by looking in the nationalities of the authors, the publication type (e.g. journal, conference, etc), the field (e.g. sociology, genetics etc), the university and the language of publication among others. This is because we do not want to favour any particular field, university or language when attributing policy citations.

## The findings, some limitations and iterations for the future

When we did the analysis, there was no sign of bias in any of the groups we had chosen because the difference in true positive rate was negligible. There was a noticeable difference of ~3% favouring English publications which is not very surprising given that most of the training data were in English. We did not consider this difference was important to act upon, but the fact that we could detect it was a positive sign.

There are more things that we would like to review and improve in our evaluations of fairness. For example, the dataset on top of which we did the review was not independent nor balanced for the groups we were testing which means that our estimates for some groups are based on very small sample sizes. At the same time the dataset had already been ‘seen’ by the algorithm when we were training it. In future, it is more appropriate to calculate our metrics on an independent test set. We have also only evaluated one component of our machine learning pipeline and we know that attributing citations is a complex system that would be better evaluated end to end. One other thing to note is that evaluating some groups might require us to use yet another machine learning model to predict something like language, which would then need to be evaluated independently for its own biases to make sure we are reaching the right conclusions.

We are still in the early days of working out the details of how to embed ethics in an agile team in order to reduce the risk of negative consequences from the digital products we built but we feel we have learned a lot just by devoting the time and energy to think and do this. We will be sharing more from this journey as we refine these processes and do more evaluations. In the meantime if you have any questions or comments do not hesitate to get in touch.