---
layout: post
current: post
cover: assets/images/dogs/dogs-collage.jpg
navigation: True
date: 2019-10-14
tags:
class: post-template
subclass: 'post'
title: How to train a fastai model and run it in the browser
author: davidpfahler
description: >
  End-to-end example of training a machine learning model in PyTorch with fastai, export it to ONNX and run it in the browser.
summary: >
  In-depth end-to-end example of how to train a machine learnin model in PyTorch with fastai, export it to ONNX and run it entirely in the browser in a React app.
---

This post covers an end-to-end example project of training a resnet model with [fastai](https://www.fast.ai) and [PyTorch](https://pytorch.org/), exporting it to [ONNX](https://onnx.ai/) and running it in the browser inside a [React.js](https://reactjs.org) app.

> Try the [demo](http://davidpfahler.github.io/react-ml-app)!

Beginner-friendly tutorials for [training a deep learning model with fast.ai](https://course.fast.ai), [exporting a PyTorch model to ONNX](https://pytorch.org/docs/master/onnx.html) or [creating a frontend web app with React.js](https://reactjs.org/tutorial/tutorial.html) are widely available, but when it comes to combining these different steps into a real project, information is more scarce. Hence, the goal of this post (and the accompanying [notebook](https://github.com/davidpfahler/react-ml-app/blob/master/train_dog_classifier_with_fastai_export_to_ONNX.ipynb) and [repository](https://github.com/davidpfahler/react-ml-app)) is to close this gap and show you how the training, export and deployment of a (close to) state-of-the-art deep learning model that runs in any modern web browser with a useable user interface comes together.

## Train a resnet with fastai and export to ONNX

This post assumes that you are familiar with training a model using fastai in general. If you are new to fastai, I wholeheartedly recommend their [free MOOC](https://course.fast.ai). The training process used is described in and can be reproduced with [this jupyter notebook](https://github.com/davidpfahler/react-ml-app/blob/master/train_dog_classifier_with_fastai_export_to_ONNX.ipynb). The notebook also includes the export to ONNX, which is a [built-in feature of PyTorch](https://pytorch.org/docs/master/onnx.html).

## Run the model in the browser

Deployment of the model to the browser also requires a web app that accepts the input data – in this case an image – and displays the predictions. You can quickly create a react app using [`create-react-app`](https://create-react-app.dev/). To get a somewhat nice looking UI going, this project uses the [MATERIAL-UI framework](https://material-ui.com), which provides a lot of useful components. You can find the full source code of this project in the [project's GitHub repository](https://github.com/davidpfahler/react-ml-app).

Loading and running the model is achieved with [onnxjs](https://github.com/microsoft/onnxjs/). Here are some tips when using onnxjs:

### Model warm-up

While you [load the model](https://github.com/davidpfahler/react-ml-app/blob/68bcc5005d9e0de390602fd0cdb9676d6d6da341/src/components/utils.js#L37), you most likely want to display a loading indicator to the user. I would recommend you also [warm-up the model](https://github.com/davidpfahler/react-ml-app/blob/68bcc5005d9e0de390602fd0cdb9676d6d6da341/src/components/utils.js#L27-L33) right after loading and keep the loading indicator shown to the user. The warm-up is simply the very first forward pass (i.e. inference) on the loaded model. This will make the inference much faster the second time. So for a smooth user experience, it can make sense to load and warm-up at the start so that running the model is subsequently much faster. Warm-up can be performed using a [random input tensor](https://github.com/davidpfahler/react-ml-app/blob/68bcc5005d9e0de390602fd0cdb9676d6d6da341/src/components/utils.js#L29).

### Loading the input image tensor

Getting an input image into a tensor format is somewhat tricky in the browser. I use an HTML dropzone, which can handle drag & dropped files as well as a regular file choser dialog. If you are using react, I recommend the [react-dropzone](https://github.com/react-dropzone/react-dropzone) component. Once an image file is dropped there, you will need to [load the image file to a `canvas`](https://github.com/react-dropzone/react-dropzone), which seems to be the only way to get the raw image data from an encoded file like a JPG in JavaScript.

### Preprocessing the input tensor

Make sure that you are [normalizing the input tensor](https://github.com/davidpfahler/react-ml-app/blob/68bcc5005d9e0de390602fd0cdb9676d6d6da341/src/components/utils.js#L54-L73) using the same statistics that you used when training your model. In this case, these were the `imagenet_stats`.

### Interpreting the `outputMap`

Running the model asynchronously returns an array of numbers, which has the same length as your classifier has classes. Each number thus representens the activation for a given class. To interpret this result, I selected only the top five activations and then [computed a percentage "probability"](https://github.com/davidpfahler/react-ml-app/blob/master/src/components/Predictions.js#L6-L24). Note that this isn't exactly scientific, but gives the user an idea of how confident the model is when making a prediction.

## Demo time: Try it out!

Try out [the live demo version of the dog breed classifier](http://davidpfahler.github.io/react-ml-app)! It should work in all modern browsers, but depending on the hardware, especially on low-end phones, it might not be able to load or run the model. Be aware that the model itself is about 48MB is size, so you probably want to have a good connection.

The best part: none of the images you give to the classifer ever leave your device! All of the inference is done locally, in your browser.

## Feedback welcome

I could and want to say a lot more about the joys and pains of making this little project, but I also want to get this information out there as soon as possible. So if you have any specific questions, suggestions or recommendations, please do not hesitate to contact me on twitter [@davidpfahler](https://twitter.com/davidpfahler). I almost certainly made mistakes in the implementation or explanation, so if you find any, please let me know!

## Acknowledgements

I want to thank first and foremost the amazing team at fast.ai and the community at the [fast.ai forums](https://forums.fast.ai), who are always most helpful. This little pet project obiously stands on the shoulders of (open-source) giants. Thanks to all of them!