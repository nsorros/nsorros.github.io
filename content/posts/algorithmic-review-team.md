---
title: "Doing algorithmic review as a team: a practical guidance"
date: 2021-03-24T10:10:30+03:00
tags: ["machine learning", "fairness"]
---

Products that rely on data science run the risk of incorporating societal biases in the algorithms that power them — potentially causing unintended harms to their users. At Wellcome Data Labs we have been thinking and openly sharing our journey towards surfacing and mitigating those harms. We split our review process in two streams

1. Product impact analysis which looks into harms that can be caused by the product irrespective of the algorithm
2. Algorithmic review process which looks into harms introduced by the model or data irrespective of the product

We have written about product impact analysis in the [past](https://medium.com/wellcome-data-labs/doing-impact-analysis-as-a-team-practical-guidance-f68f0427b4e1) so this post is going to discuss the second part of our review process in more detail.

&nbsp;
![ethics-product-development](/images/ethics-product-development.png#center)
&nbsp;

## Why do algorithmic review?

The goals of the algorithmic review are twofold

- Engage data scientists and the product team in thinking about the potential harms that a machine learning model or algorithmic process in general can cause
- Facilitate a discussion between the technical and non technical parts of the product team from which different views will surface around the impact certain technical decisions have on the users of the product

## Who is involved?

The algorithmic review, as a more technical kind of review, is primarily completed by the data science team. Certain aspects of the review are about ensuring particular steps were completed e.g. “Is there a plan to protect and secure data?”. These points require less discussion and are more clear cut. Other areas though, most notably agreeing what [fairness metric](https://towardsdatascience.com/a-tutorial-on-fairness-in-machine-learning-3ff8ba1040cb) will be used and which groups of users will be evaluated, require careful thought and engagement from the entire product team and, sometimes, subject matter experts outside the team.

In terms of roles, the product manager is responsible for ensuring that the review happens and about the steps taken to mitigate certain risks. In organisations where data scientists are not embedded in product teams, that role falls to their respective manager. We also recommend that a member of the technical team takes the role of the fairness lead while a member of the non technical team assumes the role of ethics lead.

## When to do it?

The review should happen every time the development of a new algorithm begins or before it is deployed and used by users. The benefit of doing the review early is that you can identify harmful issues early. On the other hand, the later you do the review the more information there is available for the actual algorithm being evaluated. The review should also happen in every iteration of the algorithm.

## What is involved?

At the very least the completion of a checklist (see below) to ensure that different areas that can cause harm have been considered. Often a fairness review follows where a fairness metric is calculated for all relevant groups to investigate potential bias. The process of completing unchecked items to reduce risk is managed by the product owner.

&nbsp;
![deon-checklist](/images/deon.png#center)
&nbsp;

## 1. Complete checklist

The first step is for the data scientist to complete a pre agreed checklist which covers most areas that need to be scrutinised. We recommend using the [DEON](https://deon.drivendata.org/) checklist.

The checklist will ideally live in the same place as the code e.g. Github and will be reviewed in the same way the code is being reviewed in the team e.g. Pull request. The review should include the fairness lead in most items. For certain items it should trigger a discussion with the ethics lead if not the entire product team (see next section).

For the items that are left unchecked, actions need to be agreed with the fairness lead and prioritised for completion from the product manager.

## 2. Discussing items

Certain items in the checklist will undoubtedly need multi disciplinary approval. This is normal and expected. For example deciding whether the data were representative or what metric should be used to evaluate fairness is not a decision that a data scientist could or should make on their own. This is where the ethics lead comes in and either recommends a wider team discussion or agrees on best actions with the data scientist and fairness lead. For example in the DEON checklist we would recommend ensuring that the following items are always discussed with the ethics lead:

- *C.1 Missing perspectives*: Have we sought to address blindspots in the analysis through engagement with relevant stakeholders (e.g., checking assumptions and discussing implications with affected communities and subject matter experts)?
- *C.2 Dataset bias*: Have we examined the data for possible sources of bias and taken steps to mitigate or address these biases (e.g., stereotype perpetuation, confirmation bias, imbalanced classes, or omitted confounding variables)?
- *D.1 Proxy discrimination*: Have we ensured that the model does not rely on variables or proxies for variables that are unfairly discriminatory?
- *D.2 Fairness across groups*: Have we tested model results for fairness with respect to different affected groups (e.g., tested for disparate error rates)?
- *D.3 Metric selection*: Have we considered the effects of optimizing for our defined metrics and considered additional metrics?
- *D.5 Communicate bias*: Have we communicated the shortcomings, limitations, and biases of the model to relevant stakeholders in ways that can be generally understood?

## 3. Fairness review

Even though it is not the only thing that avoids harm, a fairness review is at the heart of the algorithmic review. So much so that when we started doing algorithmic reviews we almost exclusively focused on this part before later on extending to more items and using a checklist approach. Core to the fairness review process is defining the metric of interest as well as the groups that will be evaluated.

Deciding which metric and groups will be used should involve the entire product team as it is highly subjective and application specific. In our case, most of our work involves researchers and the grants they submit, so useful groups for us are the language (e.g. English, French) and the thematic topic (e.g. Neuroscience) of the grant as well as the stage of the researcher’s career (e.g. early/senior).

Making a decision around what fairness metric should be used is slightly more challenging as there is a real tradeoff to be made, false positives vs false negatives, and no right answer. As an example in a cancer diagnosis test, a false positive might trigger an unnecessary and hurtful treatment but a false negative would risk missing the cancer. Unfortunately there is often [no way to optimise for more than one metric](https://arxiv.org/pdf/1609.05807.pdf).

## 4. Completing items

When items are completed from the data scientist, the checklist should be updated and reviewed again. The data scientist should provide information about the steps taken to complete the action. This justification needs to be approved by both the fairness and ethics lead for an item to be considered complete.

## Steps in summary

1. Complete DEON checklist https://deon.drivendata.org
2. Discuss with fairness lead the items on the checklist to come to a mutual agreement about which items should and should not be checked. Ensure that C1, C2 and D1, D2, D3, D5 are not checked if not discussed with ethics and fairness advocates and product manager
3. Agree and prioritise actions, such as perform a fairness review, for unchecked items with fairness lead.
4. Trigger a review e.g. create Github pull request, to incorporate the checklist and actions to the code repository of the project.
5. Product manager should prioritise and assign tasks related to the actions
6. To ensure an action is completed you need to have a discussion and sign off from fairness, ethics lead and the product manager.
7. When an action is completed, alter the checklist, providing justification or analysis for doing so. The change should be reviewed and approved in the same way e.g. pull request by both ethics and fairness leads.

## Final words

I hope you found the post interesting and the process clear. If you work in a team that is doing algorithmic or fairness review, I am curious to hear back on your process. If on the other hand you are part of a team that is interested in reviewing its algorithms and models, I hope you got some ideas on how to do it. If anything was unclear or you want to learn more about our work on the topic, do not hesitate to contact me.