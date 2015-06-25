---
layout: post
title: "Scottish Deep Learning Workshop 2015"
excerpt: "Summary of talks from the Second Scottish Deep Learning Workshop."
tags: [deep learning, academic]
comments: true
published: true
---

Earlier this month, I took a trip to Edinburgh to attend the <a
href="http://workshops.inf.ed.ac.uk/deep/deep2015/">Second Scottish
Deep Learning Workshop</a>. As well as an opportunity to see some
great talks, for me this was a chance to return to the Informatics
Forum, something I hadn't done since I graduated from my PhD
in 2010. It was a somewhat nostalgic experience to return to the city
where I spent almost half my time in the UK, made all the more so by
the incredibly sunny day that was in it. I was saddened to see that
Susie's Wholefood Diner had closed (one of my main dinner locations on
the evenings I stayed late in the Informatics Forum) but was pleased I
could still get a baked potato with vegetarian haggis on Cockburn St
before I got the train back to London.

<figure>
<a href="/images/thealmamater.jpg"><img src="/images/thealmamater.jpg"></a>
    <figcaption>The Alma Mater</figcaption>
</figure>

### The Talks

All the talks were excellent: the organisers deserve credit for
putting together such a strong program. Here, I'll focus on the ones
that seem closest to my current interests.

#### Deep Gaussian Processes

Neil Lawrence started the day with a talk on Deep Gaussian
Processes. I've seen Gaussian Processes presented a couple of times
before and been a little bit mystified but this time it felt like it
really clicked. Neil motivated Deep Gaussian Process Models by
starting with a familiar model, the multi-layer neural network, and
then showing how we arrive at the Gaussian Process by placing a prior
on the (potentially infinite) set of weights and integrating them out
to obtain a non-parametric probabilistic model. This Bayesian
treatment means a Deep Gaussian Process is less likely to over-fit (in
comparison to most other Deep Learning models) and can be trained on
relatively small data sets (again in contrast with most other Deep
Learning models). The Gaussian Process model, and hence the
complexity, is determined by the covariance matrix of the input,
resulting in training time \\( O(n^3) \\) and prediction time \\(
O(n^2) \\). This is clearly impractical for large datasets and Neil
and others have worked to reduce this. A variational approximation to
the log-likelihood (which is somehow analogous to taking a low rank
approximation to the covariance matrix) can reduce the complexities to
\\( O(nm^2) \\) and \\( O(nm) \\), where \\( m \\) is the number of
inducing variables used in the approximation. Deep generative models
have been gaining a lot of traction recently, particularly as people
try to tackle problems where there isn't an abundance of labelled
training data, and it feels like Deep Gassian Processes are a valid
alternative to the approaches based on Deep Neural Networks (e.g. <a
href="http://papers.nips.cc/paper/5352-semi-supervised-learning-with-deep-generative-models">Kingma
et al</a>). This is a topic which I am only just getting interested
in myself, but definitely something that feels worthy of further study.
Paper <a
href="http://jmlr.org/proceedings/papers/v31/damianou13a.html">here</a>

#### Teaching Machines to Read and Comprehend

Next up, Phil Blunsom described efforts at Google DeepMind to build
models for machine reading: the model "reads" a question and a
document which contains the answer to the question and is then able to
answer the question! One of the big problems encountered when training
models like this is a lack of large sets of good training data. To get
around this, Blunsom et al. have synthesised a data set by scraping
articles from CNN and the Daily Mail. Each article is paired with a
number of summary sentences and these are transformed into *Cloze*
questions by removing a single entity from the summary sentence and
asking the model to identify the missing entity. To make the task as
hard as possible, the entities are anonymised so that each entity is
replaced by a random string consistent within a single document-query
pair, but varying across different documents.  This ensures the
performance of the model on the task is not connected with it learning
background information about the corpus, e.g. when "cures X" appears
in a Daily Mail article there is a very high probability that
X=cancer.

The model they used seemed quite novel to me, combining a
bidirectional LSTM recurrent neural network with an attention
mechanism that was effective at extracting the relevant words from a
large document. A single-layer bidirectional LSTM is used to compute a
vector for the query, \\( v_Q \\), and to compute a vector for each
word in the document. The so-called attention mechanism is used to
assign a weight to each word in the document based on its relevance to
the query, which are then used to construct a vector representation
for the document, \\( v_D \\), as a weighted sum of word
vectors. Finally, \\( v_D \\) and \\( v_Q \\) are used to compute the
answer to the query. Even though it only has a single layer, this
model is able to outperform a deep LSTM, suggesting that determining
the relevant words in a document (via the attention mechanism) could
be more important than constructing higher-level features. There has
been a lot of recent work on the problem of computing a feature vector
for a document *D* that can then be used to answer queries about the
contents of *D* and this seems (to me) to be the most flexible
approach so far.  Paper <a href="here
http://arxiv.org/abs/1506.03340">here</a>

#### Do Deep Networks Really Need to be Deep?

After lunch, Rich Caruana spoke about the surprising phenomenon where
one can train a shallow network to match the performance of a deep
network. The motivation for this work is the fact that we know the
single layer neural network is a universal approximator, so it should
be sufficient to learn any discriminative function or class
probability distribution for a labelled data set. Despite this
theoretical result, in pretty much every real problem a deep neural
network has out-performed all shallow networks. This could be for a
number of reasons that people don't fully understand but one
conjecture seems to be that we are better at regularising deep
networks so can avoid over-fitting to the data distribution as we
increase the number of parameters to better approximate it. Rich
Caruana (and others including Geoff Hinton) have observed that for
several well-known benchmarks we can train a shallow network to mimic
a deep network and get similar (and sometimes better)
results. Moreover, in each of these cases just training a shallow
network on the data is not sufficient: people have only been able to
obtain comparable results when the shallow network is trained to mimic
a deep network. Rich did a really good job of explaining why this
might be the case, which I've tried summarised here:

*   The larger model corrects bad labels in the data. If our deep
    network achieves good generalisation then we should expect it to
    assign low probability to bad labels, so a shallow network trained
    to match the deep network's distribution will not be impacted by
    the bad labels.
*   More generally, the shallow model will inherit any regularisation
    from the deep model and potentially provide a second opportunity
    to regularise. Over-fitting occurs when our model learns
    properties of the training data distribution that disagree with
    the true distribution. If we have been able to avoid capturing
    these unrepresentative regularities in the training data by using
    some clever regularisation in our deep network, then the shallow
    model won't over-fit, no matter how long we train it for.
*   It is possible that stochastic gradient descent gets stuck in
    local optima more easily when training a shallow network. By first
    training a deep network, which we believe has a better chance of
    missing the sub-optimal local optima, and then copying this
    network's output we have a better algorithm for training shallow
    networks that will avoid the local optima .
*   We can generate an arbitrarily large training set for the shallow
    network by running the deep network on unobserved input
*   Soft targets, i.e. probabilities, contain more information the
    hard targets in our training set

The fact that you can do this has huge practical implications,
allowing people to train the biggest most powerful model they can, but
then compress it down into a model that is small and fast enough to be
deployed in production. Right now, the speech recognition models used
at Microsoft, and probably elsewhere too, all use this technique to
squeeze the models down to a size that will be acceptable for
production use. I spent almost 3 years at <a
href="http://swiftkey.com/">SwiftKey</a> working on the machine
learning based engine that powers their keyboard so am familiar with
the tight constraints encountered when we try to use machine learning
in a resource-constrained environment. It feels like the flexibility
of neural networks could allow us set our memory and computation
constraints up front and then train the best neural network that fits
within those constraints. Paper <a
href="http://papers.nips.cc/paper/5484-do-deep-nets-really-need-to-be-deep">here</a>

#### The Rest

Iain Murray spoke about applications of density estimation to
cosmology, yet another case of ML trying to come in and trump all
existing methods for a computational problem in another field. One
interesting aside in this talk was in regard to the use of RBMs to
generate features for supervised learning. Iain developed a method for
estimating the quality of the density estimation achieved by an RBM
and ranked three different models according to how well they estimated
the underlying data distribution. He then used the RBMs to construct
features for a classification problem and ranked the models according
to the accuracy of the classifier. Surprisingly, the ranking was
reversed, i.e. the better a model was at capturing the underlying data
distribution the worse the accuracy of the classifier.

Pawel Swietojanski on adapting neural models to the speaker for speech
recognition problems, a subject which resonates with me from my
SwiftKey days. One of the challenges of things like speech recognition
or next word prediction is that each user will have their own specific
style that the model needs to capture and combine with a model trained
on the wider background data. This used to be a very hard thing to get
right but it seems that the flexibility of neural networks are
well-suited to this kind of problem. 

Michael Pfeiffer gave a talk on Deep Spiking Networks. My
computational neuroscience background is too sparse to be able to make
an informed assessment of this last talk, but for me the take home
message was that this type of model can be implemented asynchronously,
on special hardware, and can smoothly trade-off running time against
prediction quality. This means that as well as being more biologically
plausible models of the brain, they also give rise to special-purpose
implementations which can be used to get deep neural networks onto
embedded devices.... Much like what's inside our heads is a
special-purpose "wetware" implementation for our very own embedded
device!

Alex Graves finished the day with his work on the Neural Turing
Machine, a complicated model that equips a recurrent model with a
separate memory so it can learn to perform complex sequential
tasks. This model feels a bit complicated right now, with many
moving parts, and there are an increasing number of papers appearing
that try to develop simpler ways to equip a neural network with an
external memory. I don't think these models are truly practical yet
but it's certainly one of the more interesting and ambitious lines of
research in machine learning and one worth keeping up with.
