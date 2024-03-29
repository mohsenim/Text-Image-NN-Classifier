# Text Analysis Using Deep Neural Models Trained for Image Processing

Visualization of information eases perception and interpretation of phenomena. However, when it comes to texts, which have a temporal dimension and contain discrete units, it becomes challenging to represent them visually. Nevertheless, there are methods that can be used to visualize texts, even very long ones. 

Another advantage of converting text into images is that deep neural models pre-trained on millions of images which have shown impressive results in various applications, can be used to analyze text effectively. I will discuss this topic in two parts:

## 1- [Text classification Using Pre-Trained Deep Neural Models for Image Classification](./text_to_image__classification.md) 
This section focuses on long text classification using deep neural models trained for image processing. I will explain how texts can be converted into series of text properties, which can then be transformed into images using Markov Transition Field. These generated images can subsequently be analyzed using powerful image classifiers to identify the category of texts. 

[link to the text](./text_to_image__classification.md)

## 2- [Fractality Analysis of Converted Text to Images Using Deep Neural Models](./text_to_image__fractality_regression.md) 
Images generated from text property series using Markov Transition Field appear to exhibit self-similar and fractal-like patterns. In this part, I investigate whether these images preserve fractal features. To address this, I define a regression problem and fine-tune a pre-trained deep neural model to predict the degree of fractality of texts.

[link to the text](./ext_to_image__fractality_regression.md)
