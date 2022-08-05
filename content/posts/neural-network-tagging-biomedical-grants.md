---
title: "A neural network tagging biomedical grants"
date: 2020-11-24T10:10:30+03:00
tags: ["machine learning", "neural networks", "cnn"]
---
![meshcnn-architecture](/images/meshcnn-architecture.png#center)
&nbsp;

Neural networks have been a ubiquitous part of the resurgence of Artificial Intelligence over the last few years. Unsurprisingly then, we decided to use a neural network as the modelling approach for tagging our grants with [MeSH](https://en.wikipedia.org/wiki/Medical_Subject_Headings). Neural networks have raised the state of the art performance on the task to 71% from below 60%. Understandably neural networks may feel complicated to someone outside the field of machine learning, but in this piece my goal is to make them as understandable as logistic regression and Principal Component Analysis (PCA). The dataset we use is a collection of 8M publications from PubMed that are already assigned MeSH tags manually by expert annotators.

&nbsp;
![meshcnn-fnn](/images/meshcnn-fnn.png#center)
&nbsp;

The simplest neural network is logistic regression itself. In fact the predict component of our neural network is not very different from a logistic regression model, so let’s start with that. Our goal is to classify (predict) an incoming grant with one of the available MeSH tags, e.g. ‘Cancer’ or ‘Infectious diseases’. As such, the inputs of the model are the words mentioned in the grant abstract and the outputs are the mesh tags associated with the grant. To create a mathematical model we first need to convert both input and outputs into numerical representations in the form of vectors. The output vector is of equal size to the number of mesh tags the model can recognise and contains a 1 for every mesh tag that is associated with a grant. For the input, the simplest way to represent the words would be to create a vector equal to the total number of unique words in all of our grants, we call that the vocabulary. Each word in the grant abstract would be then represented by a big vector which is zero everywhere but the element that corresponds to that word where the value would be 1. We can combine those words together to create one representation for the grant by summing or averaging the word vectors.

&nbsp;
![meshcnn-embedding](/images/meshcnn-embedding.png#center)
&nbsp;

Assuming a vocabulary of 100K words then each MeSH tag is a logistic regression model of 100K parameters. These parameters give different weights to each word and together they combine into a probability that a tag is relevant. The disease part of MeSH that we use contains approximately 5k tags, so we have 5K logistic regressions each with 100k parameters, so in total 500M parameters. In fact, this is very close to our baseline method which is 57% accurate in this problem. One key difference with our baseline model, which is based on TF IDF-SVM, is the size of the word vectors. It turns out that instead of representing the words as a large vector equal to the vocabulary size, in our example 100k, with all zeros but the index of the vector that represents the word, we can instead condense that representation to a much smaller size of say 400 that contains mostly non zero terms. We call the former one-hot representation while the latter dense or distributed representation.

&nbsp;
![meshcnn-embedding](/images/meshcnn-embedding2.png#center)
Left: Transforming one hot representations to dense representations through 400 logistic regression. Right: Embeddings matrix that acts as a lookup table for dense representation for each word in the vocabulary
&nbsp;

There are a couple of ways we can condense the vector without losing useful information. For example we could stack all grants of one-hot vectors into a big matrix and perform dimensionality reduction (e.g. via PCA). We could also apply dimensionality reduction to the words co-occurrence matrix, which in our case would be a 100K by 100K matrix with each value corresponding to how often two words appear together (e.g. within 3 words of each other). In neural networks the same effect can be achieved by learning a 400 vector representation for each word using 400 logistic regressions with 100k parameters each. Together the vector representations for each word create a lookup table we refer to as an embeddings matrix. Regardless of the implementation details, this process creates word representations that have been shown to perform better in various problems and come with some nice properties such as: words that are semantically similar are closer in space than words that have completely different meanings. This transformation from one hot representations to dense representations increases the performance of the model from 57% to 60%.

&nbsp;
![meshcnn-hiddenlayer](/images/meshcnn-hiddenlayer.png#center)
In red, the hidden layer. Each item in the hidden layer is a logistic regression that operates on the sum of word vectors. Each mesh tag in the output vector is also a logistic regression that connects to all items of the hidden vector.
&nbsp;

Another difference of the actual predict module is that instead of directly linking the word vectors to mesh tag predictions there is an intermediate layer which in neural network terminology we refer to as the hidden layer. This hidden layer increases the representational power of the model and it works in exactly the same way as the logistic regressions we described. Two other smaller differences are firstly that the function applied to all layers but the last is not the logistic function but a RELU function and that instead of summing we average the word vectors both of which are standard defaults that perform well across problems but are not that consequential in this problem.

&nbsp;
![meshcnn-residual](/images/meshcnn-residual.png#center)
&nbsp;

One of the key characteristics of a language and as a result of the text that is the input to our model is that the order of words matters. Until now we have been mainly treating words irrespective of their order, since soon after their transformation into one hot or dense vectors we were summing or averaging them. This approach is called ‘bag of words’ and it works surprisingly well as we saw with our results so far. Unlike the bag of words approach, the first layer of our neural network retains information about the order of the words. The way we do that is by using a one dimensional (1D) convolution (another common way is to use a unidirectional or bidirectional LSTM). We can stack multiple of these convolutional layers to increase the representation power of the model or capacity in machine learning terminology. Finally the 1D convolution is followed by a layer normalisation and a residual connection which is the line that feeds the input to the output. All we need to know for both is that their addition helps the learning process of the parameters for all those logistic regressions.
stack of 4 convolution layers. to maintain the same dimensionality two zero word vectors are added at the start and end, depicted in dotted reds.

&nbsp;
![meshcnn-convolution](/images/meshcnn-convolution.png#center)
&nbsp;

A 1D convolution is nothing more than another logistic regression that combines n word vectors into a new representation. The number of words that will be combined is called the context window as it defines how much context we will combine into the new representation, let’s assume the context window is 3. Stacking convolutional layers increases the context that is being encoded in the new representations so for example stacking 4 of these modules allows each new representation to encode the meaning of 7 words in total. Since each 1D convolution combines 3 word vectors to one we also add two zero vectors at the start and end to the same number of vectors in the output. Stacking four of those modules increases the performance of the model to 64%.

&nbsp;
![meshcnn-attention](/images/meshcnn-attention.png#center)
&nbsp;

The last component to understand is hierarchical attention. Attention is one of the latest neural network techniques that has been proven quite effective, especially applied in text problems. The idea behind attention is that for each output different parts of the text might be important. The two components that we described so far either combine all words into one representation, or combine nearby words into one representation but do not allow for a custom subset of all words to be part of the new representation. This is exactly what attention does. It does this by introducing one vector of equal dimension to the word which gets multiplied by each word to give a similarity or importance score to each word. Then it recombines the word vectors according to that score. Since the attention mechanism received n words and returns one vector, to keep the same dimensionality we add n of those vectors where n the size of the input vector. As it turns out, contrary to prior work this attention mechanism does not seem to increase the performance of the model so we have made it optional.

This should give an idea of how the information flows into the neural network and the justification behind each component. You might be wondering at this point how does this neural network or model learn to tag MeSH in the first place. The main idea for training machine learning models including neural networks is that we provide the model with a lot of examples with different mesh tags and change the parameters so that the model slowly but steadily converges towards those parameters that perform the best. Those parameters are the different logistic regression weights we described. In technical jargon we use stochastic gradient descent and back-propagation but explaining these are out of scope and it is not important to understand the model. Rest assured though that understanding them is equally easy, or hard depending how you found the explanations above, so the magic is not hidden behind the things not explained.

Further improving accuracy involves speaking a bit about the threshold above which we assign a tag as well as how to combine different models together. These tricks can further improve performance to 68% but let’s talk about them in a follow up post.

One key challenge of this project has been scale. It currently takes 16 days on 1 Graphics Processing Unit (GPU) to train the model which is 2.5GB in size and a total of 200M parameters. The scale of the problem is the reason why it has also been difficult to apply state of the art techniques such as BERT. Speaking about BERT and the scale of this problem will also be delayed for a future post. Stay tuned.