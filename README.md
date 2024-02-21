# Text classification Using Deep Neural Image Classifiers

In this post, I explore the feasibility of conducting long text classification using a pre-trained image processing model. I will explain how to convert a very long text into a time series and subsequently transform it into an image. These images can then be analyzed using powerful image classifiers to identify the category of the text.

In what follows, I will first discuss why converting text into a time series is intriguing. Then, I explain rationale for converting text into an image, and following that, I describe a method to do so.  Finally, I will merge these components and conduct an experiment for classification of canonical and non-canonical texts.

## Time Series Representation of Text
Representation of texts in the form of time series, or more precisely the sequence of text properties, keeps the structural features, hence, allows analyzing the organization of text. This is especially important for analyzing long texts, such as literary texts, which are usually long and structure plays an important role in how those texts are perceived by the reader. Despite the importance of structure in analyzing text, the common approach in analyzing texts has been bag-of-word (or bag-of-features). There are however exceptions such as traditional probabilistic language models or more modern LSTM neural models.

One simple way to convert text into a time series is by constructing the sequence of sentence lengths, as the sentence is one of the most fundamental linguistic levels. Previous studies have shown that sentence length, despite its straightforwardness, reveals interesting information about the structure of texts. Human-written texts exhibit fractality in their global structure, and fractal features can be used to classify text categories (see, for example, Drozdz et. al, 2016 and Mohseni et. al, 2021).

In the following experiment, I also used sentence length to represent long fictional texts. Time series of sentence lengths are converted to images using Markov Transformation Field which I turn to in the next section.

## Converting Time Series into Images
The time-intrinsic aspect of texts and the discrete structure of textual units have likely been two of the main obstacles in text processing. If we could visualize textual information like an image, we could more effectively grasp it and analyze text. Fortunately, there are methods for converting time series to images. So, if we represent a text by time series as I explained in the previous section, we can utilize these methods to visualize text in the form of images. One of these methods is the Markov Transition Field (MTF) technique, introduced by Wang and Oates (2015) to convert a time series into an image using transition probabilities between elements of the series.

Give a time series $X=x_1, x_2, \cdots, x_n$ and quantiles $Q=q_1, q_2, \cdots, q_m$, MTF processes the series as follows:

1-	 Assign $x_i, i=1, \cdots, n$ to the corresponding bins $q_j$.

2-	 Calculate the transition probability matrix $P_{m\times m}$, where each entry $p_{i,j}$ represents the probability of transition from $q_i$ to $q_j$. This can be achieved by counting the number of transitions to $q_j$ from all quantiles and then normalizing the values so that $\sum_i p_{ij}=1$. Thus far, transition probabilities have been calculated, but temporal information of the series has been lost.

3-  To preserve the temporal information alongside the probabilities, construct the Markov transition field matrix $M_{n \times n}$ (where $n$ is the length of the series). Each entry $p_{i,j}, 1<i,j<n$ of the matrix represents the probability of transition from $q_i$ (the corresponding quantile of $x_i$) to $q_j$ (the corresponding quantile of $x_j$) with a distance of $i-j$ steps (where $|i-j|$ is the distance between the two values $x_i$ and $x_j$ in the time series).

4-  Visualize the MTF matrix $M_{n\times n}$ in the form of an image. 
The number of quantiles, $m$, can be set based on the range of values and the desired granularity.

### A Toy Example
Before presenting real examples, I will illustrate MTF using a toy example. I used the `pyts` package for Python and the sample code available on its website to convert a time series of the `cos` function for the interval $[\pi/2, 7\pi/2]$ into an image, as shown in the following plot. The number of quantiles was set to 10.

![example_cos](./fig/example_cos.png)


Let’s focus on the diagonal of the image first, which reveals local fluctuations in the time series. As depicted in the plot, the cos function is compressed around $2\pi$, and values near this point exhibit the highest proximity to each other. Consequently, the MTF image displays a large square corresponding to this region, with squares diminishing in size as we move away from this point. Similarly, around $\pi$ and $3\pi$, the function condenses, resulting in high probabilities. However, transitions between values further from an integer multiple of $\pi$ occur more rapidly, leading to lower probabilities (represented by smaller red squares) in the image.

The image also shows some high probabilities (big red squares) far from the diagonal. What do they exhibit? According to step 3 of the MTF procedure, $p_{i,j}$ represents the transition probability for two values with the distance $|i-j|$. Therefore, for points $i, j, |i-j|\gg0$, the image represents non-local fluctuations. In the case of the `cos` function, the probability of transition from values near $\pi$ to values near $3\pi$ with a distance of $2\pi$ is high, as demonstrated in the produced image.

### Real Examples

