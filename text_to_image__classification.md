# Text classification Using Pre-Trained Deep Neural Models for Image Classification

In this post, I explore the feasibility of conducting long text classification using a pre-trained image processing model. I will explain how to convert a very long text into a time series and subsequently transform it into an image. These images can then be analyzed using powerful image classifiers to identify the category of the text.

In what follows, I will first discuss why converting text into a time series is intriguing. Then, I explain rationale for converting text into an image, and following that, I describe a method to do so.  Finally, I will merge these components and conduct an experiment for classification of canonical and non-canonical texts.

## Time Series Representation of Text
Representation of texts in the form of time series, or more precisely the sequence of text properties, keeps the structural features, hence, allows analyzing the organization of text. This is especially important for analyzing long texts, such as literary texts, which are usually long and structure plays an important role in how those texts are perceived by the reader. Despite the importance of structure in analyzing text, the common approach in analyzing texts has been bag-of-word (or bag-of-features). There are however exceptions such as traditional probabilistic language models or more modern LSTM neural models.

One simple way to convert text into a time series is by constructing the sequence of sentence lengths, as the sentence is one of the most fundamental linguistic levels. Previous studies have shown that sentence length, despite its straightforwardness, reveals interesting information about the structure of texts. Human-written texts exhibit fractality in their global structure, and fractal features can be used to classify text categories (see, for example, Drozdz et. al, 2015 and Mohseni et. al, 2021).

In the following experiment, I also used sentence length to represent long fictional texts. Time series of sentence lengths are converted to images using Markov Transformation Field which I turn to in the next section.

## Converting Time Series into Images
The time-intrinsic aspect of texts and the discrete structure of textual units have likely been two of the main challenges in text processing. If we could visualize text like an image, we could more effectively grasp textual information and analyze text. Fortunately, there are methods for converting time series to images. So, if we represent a text by time series as I explained in the previous section, we can utilize these methods to visualize text in the form of images. One of these methods is the Markov Transition Field (MTF) technique, introduced by Wang and Oates (2015) to convert a time series into an image using transition probabilities between elements of the series.

Give a time series $X=x_1, x_2, \cdots, x_n$ and quantiles $Q=q_1, q_2, \cdots, q_m$, MTF processes the series as follows:

1-  Assign $x_i, i=1, \cdots, n$, to the corresponding quantile $q_j$.

2-  Calculate the transition probability matrix $P_{m\times m}$, where each entry $p_{i,j}$ represents the probability of transition from $q_i$ to $q_j$. This can be achieved by counting the number of transitions to $q_j$ from all quantiles and then normalizing the values so that $\sum_i p_{ij}=1$. Thus far, transition probabilities have been calculated, but temporal information of the series has been lost.

3-  To preserve the temporal information alongside the probabilities, construct the Markov transition field matrix $M_{n \times n}$ (where $n$ is the length of the series). Each entry $p_{i,j}, 1\le i,j\le n$, of the matrix represents the probability of transition from $q_i$ (the corresponding quantile of $x_i$) to $q_j$ (the corresponding quantile of $x_j$) with $|i-j|$ transitions ($|i-j|$ is the distance between the two values $x_i$ and $x_j$ in the time series).

4-  Visualize the MTF matrix $M_{n\times n}$ in the form of an image. 

The number of quantiles, $m$, can be set based on the range of values and the desired granularity.

### A Toy Example
Before presenting real examples, I will illustrate MTF using a toy example. I used the [`pyts`](https://pyts.readthedocs.io/en/stable/index.html) package for Python and the sample code available on its website to convert a time series of the `cos` function for the interval $[\pi/2, 7\pi/2]$ into an image, as shown in the following plot. The number of quantiles was set to 10.

![example_cos](./fig/example_cos.png)


Let’s focus on the diagonal of the image first, which reveals local fluctuations in the time series. As depicted in the plot, the `cos` function is compressed around $2\pi$, and values near this point exhibit the highest proximity to each other. Consequently, the MTF image displays a large square (high probabilities) corresponding to this region, with squares diminishing in size as we move away from this point. Similarly, around $\pi$ and $3\pi$, the function condenses, resulting in high probabilities. However, transitions between values further from an integer multiple of $\pi$ occur more rapidly, leading to lower probabilities (represented by smaller red squares) in the image.

The image also shows some high probabilities (large red squares) far from the diagonal. What do they exhibit? According to step 3 of the MTF procedure, $p_{i,j}$ represents the transition probability for two values with the distance $|i-j|$. Therefore, for points $i, j, |i-j|\gg0$, the image represents non-local fluctuations. In the case of the `cos` function, the probability of transition from values near $\pi$ to values near $3\pi$ with a distance of $2\pi$ is high, as demonstrated in the generated image.

### Real Examples
I have selected three example texts for analysis:

1-   The Golden Bowl by Henry James

2-   The Puppet Crown by Harold MacGrath

3-   A Synopsis of the Birds of North America by John James Audubon

For each of these texts, I extracted the series of sentence lengths. However, rather than directly applying MFT to the time series, I first subtract the mean of the series from each value and then compute the cumulative sum. This process converts the series into a random walk, which is referred to as the profile of the series (for more information, you can refer to [this repository](https://github.com/mohsenim/Multifractality)). The figure below shows the result.

![real_examples](./fig/real_examples.png)


The MFT image of _The Golden Bowl_ and _The Puppet Crown_ demonstrate fractal-like pattern, while the plot of _A Synopsis of the Birds of North America_ shows more noisy pattern with the probabilities dispersed all over the plot. Interestingly, in another study, as illustrated in [here](https://github.com/mohsenim/Multifractality/blob/main/MFDFA.ipynb), I showed that the first two texts exhibit fractal properties, while the last text does not.

Although my focus here is not on fractality, I conducted a simple experiment to suggest that the MTF conversion of sentence length series might be related to fractality. We know that shuffling can disrupt long-range correlations in a signal and eliminate or reduce its fractality. Therefore, I applied MFT to the time series of the aforementioned texts, but this time I first shuffled the time series. As shown in the following figure, the distribution of probability changes, and we cannot observe any clear self-similar patterns.

![real_examples](./fig/real_examples_shuffled.png)


# Classification of Texts Using Pre-Trained Neural Models for Image Classification
For this experiment I used the [JEFP Corpus](https://github.com/mohsenim/JEFP-Corpus), which contains two categories of fictional texts: canonical and non-canonical. Sentence length series for all texts were extracted and converted to images using the MFT method as explained above.
 
Now that the texts have been transformed into images, we can leverage powerful pre-trained neural networks and fine-tune them for our text classification task.
I used `ResNet-18`, a deep convolutional neural network with 18 layers trained on `ImageNet`, a vast dataset categorized into 1000 different objects. To fine-tune this model for the classification of canonical and non-canonical text, I used `fastai`.

The `JEFP` corpus contains a relatively small number of texts: 76 canonical and 130 non-canonical texts. To ensure more reliable results, I applied 5-fold cross-validation using `sklearn`. After fine-tuning `ResNet-18` and averaging the results, a mean accuracy of 70 ± 7 was obtained.

While my primary focus was to examine the feasibility of text classification using pre-trained deep neural networks for image classification, rather than achieving state-of-the-art performance in classfication of canonical and non-canonical texts, it is worth emphasizing a few points.

Firstly, the only text property used for this experiment was a very basic language component, i.e., sentence length. However, it is possible to represent texts using several text properties, convert them to images, and feed them altogether to a classification model.

Secondly, the main motivation behind converting text to images is that visualization eases perception and interpretation of information, although some information inevitably may be lost.

Finally, I discussed a possible relation of MTF images with the fractality of texts in the previous section. In Mohseni et al. (2021), it was shown that fractal features of several text properties can distinguish canonical from non-canonical texts with an accuracy of 65%. Thus, the obtained result here (70%) does not appear disappointing as it was in the beginning.

### References
* Mohseni, Mahdi,Volker Gast, and Christoph Redies (2021). “Fractality andVariability in Canonical and Non-Canonical English Fiction and in Non-Fictional Texts”. Frontiers in Psychology 12, p. 920.
* Wang, Zhiguang and Tim Oates (2015). “Imaging Time-Series to Improve Classification and Imputation”. International Joint Conference on Artificial Intelligence, pp 3939–3945.
* Drozdz, Stanislaw, Pawel Oswiecimka, Andrzej Kulig, Jaroslaw Kwapien, Katarzyna Bazarnik, Iwona Grabska-Gradzinska, Jan Rybicki and Rybicki Stanuszek (2015). "Quantifying origin and character of long-range correlations in narrative texts". Information Sciences (331), pp 32-44. 