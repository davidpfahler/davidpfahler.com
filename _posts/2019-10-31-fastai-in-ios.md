---
layout: post
current: post
cover: assets/images/dogs/dogs-coreml.jpg
navigation: True
date: 2019-10-31
tags:
class: post-template
subclass: 'post'
title: How to train a fastai model and run it on iOS
author: davidpfahler
description: >
  End-to-end example of training a machine learning model in PyTorch with fastai, export it to ONNX, convert it to CoreML and run it on iOS.
summary: >
  In-depth end-to-end example of how to train a machine learning model in PyTorch with fastai, export it to ONNX, convert it to CoreML and run it on iOS in a react-native app.
---

This post covers an end-to-end example project of training a resnet model with [fastai](https://www.fast.ai) and [PyTorch](https://pytorch.org/), converting it to [CoreML](https://developer.apple.com/documentation/coreml) and running it inside a [react-native](https://facebook.github.io/react-native/) iOS app.

> Find the [notebook](https://github.com/davidpfahler/react-native-ml-app/blob/e4abc813f2c3e7e147454afbcbb4edd14c9ffe16/train_dog_classifier_with_fastai_export_to_CoreML.ipynb) and code in the [GitHub repository](http://davidpfahler.github.io/react-ml-app)!

My [last article](/fastai-in-the-browser) covered how to train a model in fastai and convert it to ONNX to run it in the browser using onnxjs and React.js. This post goes one step further and into a slightly different direction: The goal is to create a react-native iOS app, that performs the classification of dog breeds locally on the iPhone using CoreML. If you did not read my [last article](/fastai-in-the-browser) yet, I would recommend you catch up, because the following will assume you know about the training and exporting to ONNX parts already.

## Converting the model from ONNX to CoreML

> See the notebook [section "Export to ONNX"](https://github.com/davidpfahler/react-native-ml-app/blob/e4abc813f2c3e7e147454afbcbb4edd14c9ffe16/train_dog_classifier_with_fastai_export_to_CoreML.ipynb#Export-to-ONNX) and onwards.

PyTorch has built-in support for export to ONNX. For details see [my previous blog post](/fastai-in-the-browser). Going from ONNX to CoreML requires [`coremltools`](https://github.com/apple/coremltools) and [`onnx-coreml`](https://github.com/onnx/onnx-coreml). At first glance you might think that exporting using `onnx-coreml` is simply, but at least in this particular case, it wasn't so straigt forward. Firstly, you need to keep in mind how you normalize your input. In other frameworks it is common to perform normalization before input data is given to the model. In CoreML, at least to my limited knowledge (see [this onnx-coreml issues for reference](https://github.com/onnx/onnx-coreml/issues/338)), it seems that normalization is in the input layer of the CoreML model. When we naively export a PyTorch model, it does not have such a normalization input layer.

> Tip: Use [Netron](https://github.com/lutzroeder/Netron) to visualize your model architecture.

When you convert the ONNX model to CoreML, you need to provide an `image_input_names` parameter so the model knows which layer is the input layer that receives raw image data. This parameter takes a list of strings, which in this case is just the name of the first layer. But what is that name? To find that out quickly and easily, I used [Netron](https://github.com/lutzroeder/Netron). This program allows you to visualize your ONNX network easily and displays the name of the layer when you click on it.

Back to normalization: The bias can be accomplished by providing the `preprocessing_args` to the `convert` function of `onnx-coreml`, but to scale the inputs correctly, we need to add a layer to the beginning of our (now ONNX) model. To do that we first convert the ONNX model to CoreML using `onnx-coreml` and then use `coremltools` to load and modify this CoreML model. This is explained more in-depth in [this notebook](https://colab.research.google.com/drive/1hxpSDrL3lTZC2QDvcaQzscGLOAZSTnQC#scrollTo=NFThHbIfbmHU).

## Integrating the model in a react-native iOS app

First, I tried to use the [`react-native-coreml`](https://github.com/rhdeck/react-native-coreml) native module. Unfortunately, I couldn't get it work. So I borrowed from it and the [react-native docs on Native Modules](https://facebook.github.io/react-native/docs/native-modules-ios) to create my own native module that did [exactly why and I needed](https://github.com/davidpfahler/react-native-ml-app/blob/e4abc813f2c3e7e147454afbcbb4edd14c9ffe16/src/CoreMLNativeModule.js) and no more.

One more take away: If you can, stick to Objective-C instead of using Swift via a bridge like I did. It seems like the necesary Swift libraries blow up the bundle size by about 200MB.

For the react-native app, I used [`react-native-paper`](https://github.com/callstack/react-native-paper) for a few UI elements and [`react-native-camera`](https://github.com/react-native-community/react-native-camera) to capture pictures. `react-native-camera` needs a little more effort than one would think. It exposes a very bare-bones API, which is super flexible, but does not provide a lot of convenience out-of-the-box. For example, you need to design and code your own trigger button. All that the plugin provides is the raw camera signal and APIs to take a picture or record video. Fortunately, they have [examples](https://github.com/react-native-community/react-native-camera/tree/master/examples), but you have to know to look there.

## Feedback welcome

I could and want to say a lot more about the joys and pains of making this little project, but I also want to get this information out there as soon as possible. So if you have any specific questions, suggestions or recommendations, please do not hesitate to contact me on twitter [@davidpfahler](https://twitter.com/davidpfahler). I almost certainly made mistakes in the implementation or explanation, so if you find any, please let me know!

I also created a thread on the [fastai forums](https://forums.fast.ai/t/running-a-fastai-model-in-ios-using-coreml/57553), so if you are active there, please join the discussion!

## Acknowledgements

I want to thank first and foremost the amazing team at fast.ai and the community at the [fast.ai forums](https://forums.fast.ai), who are always most helpful. This little pet project obiously stands on the shoulders of (open-source) giants. Thanks to all of them!
