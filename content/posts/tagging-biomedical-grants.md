---
title: "Tagging biomedical grants with 29K tags"
date: 2021-12-13T10:10:30+03:00
tags: ["neural networks", "multilabel classification"]
---


In a previous post we spoke about a neural architecture we developed for classifying our grants with ~5K disease tags from the MeSH (Medical subject Headings) hierarchy. In this post we will touch on the techniques needed to scale to a model to classify all ~29K MeSH tags. Our dataset consists of 14M biomedical publications labelled with one or more [MeSH](https://en.wikipedia.org/wiki/Medical_Subject_Headings) tags (on average 12 tags per publications), so the challenge is both the thousands of outputs our model needs to recognise and the millions of examples it needs to learn from. This problem is commonly referred to as extreme multilabel classification in the literature. There are four areas that require special attention in such problems
- Model size
- Memory footprint
- Training time
- Inference time

## Model size

In our last post we mentioned that the neural model we built to classify the 5K disease tags was using ~200M parameters and required 5GB of disk space. Naively scaling to 28K outputs would add an additional ~200M parameters and double the disk space. The problem gets even worse if we assume a one vs rest linear model with a 400K vocabulary for which we would require at least 400K * 28K * 8bytes (float64) ~90GB.

The most effective strategy to drastically reduce the size needed while implicitly regularizing your model is to set all parameters to zero below a threshold, say 0.1 and store the weights using a sparse matrix. This strategy scales nicely with the number of outputs as you can increase or decrease the threshold depending on your requirements. In our case with a 0.1 threshold and a linear model our model size ends up being ~2GB.

With neural models the embedding table ends up consuming a lot of space. Assuming an embedding size of 400 and a vocabulary of 400K words, the table would consist of 160M parameters alone. One way to reduce that is by using subword tokenization where the most frequent words are represented by tokens and less frequent words are broken into subwords which appear more often. Splitting rare words into subwords that are already part of the vocabulary is how the size is reduced. A typical vocabulary size using subword tokenisation is 30K, an order of magnitude less than the vocabulary of 400K we were previously using. As an example our 5GB neural model dropped to 700MB using this strategy.

&nbsp;
![subword-tokenisation](/images/subword-tokenisation.png#center)
Subword tokenization example using wordpieces. Source: https://arxiv.org/pdf/1609.08144.pdf
&nbsp;

Another strategy to reduce size, that we did not try is quantization, i.e.: using fewer bytes per parameter. For example you can reduce from 8 bytes (float64) to 1 byte (float8) which can shrink the size another 8x. We did not try this as in our case as 700MB and 2GB were acceptable sizes.

## Memory footprint

The memory needed to train such models is one of the first hurdles you need to overcome when you work on such problems. To illustrate the problem, our target variable Y, which is of size 14M examples x 28K labels, would need 49GB even if loaded into a binary matrix (1 bit per element). Similarly our feature vector X for a linear model would be 14M examples X 400K vocabulary size, so at a minimum we would need 5.6TB using 1 byte per element (float8).

Fortunately in our case both matrices are sparsely populated: the feature vector because not all words from the vocabulary appear in every example; and the target vector because not all labels are relevant. This reduces the memory requirements to approximately the number of nonzero elements. In our case we have an average of ~400 words and ~20 labels per example which translates to 5.6GB and 35MB respectively in the best case scenario (float8, binary). In fact, [TfidfVectorizer](https://scikit-learn.org/stable/modules/generated/sklearn.feature_extraction.text.TfidfVectorizer.html#sklearn.feature_extraction.text.TfidfVectorizer.transform) in scikit-learn returns a sparse vector by default and text input to neural networks is typically passed as a vector of integers referring to the vocabulary index which is also a form of sparse representation.

In practice the memory needed is higher than 5.6GB and 35MB and additionally representing data as sparse matrices only scales to a certain extent. The standard way to work with datasets that don’t fit in memory is to never load them in the first place but simply iterate through the examples and update the parameters. This is partly why stochastic gradient descent (vs gradient descent or other optimisation techniques) is so widely used in neural networks. This strategy is much more flexible as you control the size of the batches you load to work with the memory you have available.

While using sparse matrices and performing partial updates seems straightforward, in practice not all algorithms work with these strategies out of the box. For example, sklearn to date (and to my knowledge) does not support partial updates in multilabel classification using SGDClassifier (see [here](https://github.com/scikit-learn/scikit-learn/issues/8381)), whilst working with sparse target vectors in tensorflow is not straightforward.

## Training time

After you resolve any memory issues, the next challenge is the iteration cycle i.e. the time it takes to try a new idea. To a large extent reducing training time revolves around using the resources you have efficiently and using more resources as required. Cloud providers have made it relatively easy to access pretty large instances with multiple CPUs and GPUs without needing a dedicated cluster most of the time. At the same time most libraries have made it relatively easy to parallelise, for example tensorflow uses all cores by default and sklearn exposes a convenient n_jobs parameter that, when set to -1, does the same. Even training with multiple GPUs has become easier. For example in tensorflow you simply need to build your model using the context manager MirroredStrategy: a one line change. You can even scale sklearn and pandas to a cluster or to use GPUs with [Dask](https://docs.dask.org/en/stable/) and [Rapids](https://rapids.ai/) respectively.

Using the resources more efficiently is slightly trickier than scaling the resources and requires experimenting with your parameters as well as reading the documentation for the components you use to enable hidden speedups. For example it is typical to increase the batch size to the largest size the GPU memory can handle. Another example is tensorflow’s use of cuDNN which might require your parameters or data in a certain format (see [here](https://www.tensorflow.org/api_docs/python/tf/keras/layers/LSTM)).

Other than those two strategies you can also use a sample of the data to train and if you choose the sample carefully, this will have minimal effect on your performance. In multiclass classification stratified sampling to maintain the class distribution is usually enough, while in the multilabel case you ideally want to maintain the label relationship as well, see [here](https://link.springer.com/content/pdf/10.1007/978-3-642-23808-6_10.pdf). In our case, labels are highly imbalanced with many more negative examples than positive per label. In such cases it is more effective to perform a type of negative sampling. This is straightforward in the one vs rest scenario where you train one classifier per label. In that scenario you can simply use a subset of the negative examples (negative sampling) while keeping all positives. Another way would be to batch labels and keep only examples that are positive in at least one label like in the image below. In our case this reduces training time of a linear classifier from 16 days to 5 hours using 16 cores.

&nbsp;
![negative-sampling](/images/negative-sampling.png#center)
Example of negative sampling where Y is split into batches and for each batch examples with all labels zero are not used. Source: https://arxiv.org/pdf/2010.05878.pdf

## Inference time

After training a model, the final challenge has to do with inference time. The extent to which this is important also depends on whether predictions need to be available in real time or can be processed in a batch fashion. In our case we process grants and add tags once per week so inference time is not a top priority.

Improving the inference time is not very different from the considerations we mentioned for training time. You have to use your resources efficiently and you can scale them and rely on parallelisation which is even easier in this case as prediction is embarrassingly parallel.

Other than that there is yet another strategy to enable scaling in a large number of labels which is to construct a hierarchical label tree. This is similar to the hierarchical softmax trick in multiclass classification. The simplest way to leverage this strategy is to describe each label by the average of features that are relevant and then recursively cluster them to create a tree. Notice that you need to train a model for each level of a tree which is usually not a problem because the complexity is dominated by the leaves which predict the final labels. In regards to inference though, it means you don’t need to predict all labels with an O(L) complexity as in the vanilla approach, but with O(logL) which is much better. In our case log2(30000) ~ 15; so this is a 2000x improvement in inference time, with a minor cost in training time as we have to cluster the labels and train individual classifiers.

&nbsp;
![binary-tree](/images/binary-tree.png#center)
Example of a hierarchical label tree used for prediction. As we can see not all labels need to be evaluated, only those whose paths were the most probable. Source https://arxiv.org/pdf/2010.05878.pdf
&nbsp;

In our case it used to take 14 hours to run the batch job for the disease subset only (5K tags) whereas now we tag using all tags in just 5 hours, so 6x more tags in ⅓ of the time.

The model we currently use makes use of most of these techniques along with some others. It is developed by Amazon and is called PECOS, you can read more in their paper, see the code in their Github or simply pip install it from Pypi. We chose this model because, on top of implementing all these techniques which make it quite efficient, it has a well documented code and provides both a linear and a transformers based implementation with a different tradeoff of speed vs accuracy. We are currently using the linear model while experimenting with transformer based models. Our code is also open source in Github and available in Pypi.

While these techniques and considerations were presented in the context of a large multilabel classification problem they are applicable in most machine learning problems in some form. It is also definitely not an exhaustive list of what you can do. If you are working on a similar problem or have experienced similar challenges feel free to get in touch. We hope you found this post useful.