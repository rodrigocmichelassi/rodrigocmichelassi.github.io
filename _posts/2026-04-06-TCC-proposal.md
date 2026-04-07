---
title: "[TCC] Project proposal"
date: 2026-04-06 20:30:00 +/-0300
categories: [TCC, Image captioning]
description: "This post is a first proposal for my bachelor thesis. In this series of posts, I will be describing what is the thesis idea and the steps taken into consideration while developing this project, which will be an important step into my career."
tags: [machine learning, AI, ML, IQA, retina, LLM, captioning]
math: true
---

In the past few years, we have seen the Computer Science field be changed day after day in an intense pace, faster than we are able to catch up. One of the main fields affected is machine learning. It is no surprise for anyone reading this that I have been interested in machine learning for a while, and have conducted research in this field. Now, I want to take a further step into this area and adquire a broader knowledge in other deep learning areas, such as language models. This bachelor thesis comes as my first contact into developing a huge project in this revolutionary field.


### Retinal project context

Last year, I conducted a research undergraduate project involving retinal images. This was part of a broader project by the University of São Paulo Medical School (FMUSP), in which the main researchers are looking for ways to detect and prevent mental illnesses, such as cognitive decay (is this correct in english?), and one of the main sources of knowledge that may be explored for this is the human retina. The retina is interesting because it is not an invasive kind of data, and shows a large presence of blood vessels, that may be analyzed.

Given this context, last year I explored ways to use YOLO for object detection, in which we trained two models, responsible for detecting the macular region and the optic disc in retinal images. We used those informations to figure out if the images were proper for diabetic retinopathy detection, based on a famous image quality assessment protocol for retinal images. We also provided a statistical analysis of the dataset annotations, after noticing some of the data might have been mislabeled, according to this protocol.

Not only that, two other projects were conducted inside the same retinal research:
    
- Research papers comparing few-shot learning methods for retinal diseases classification
- Retinal blood vessels segmentation


### Enhancing DL for retinal diagnosis (thesis proposal)

Now that we had a look inside the project context, we can jump into my thesis proposal.

We have been pretty much daily using language models, either if it is to automatize tasks, for work effectiveness, research, summing up huge texts, even asking daily questions, this is part of everyone's life at this point. It has truly revolutionized the industry. However, research still has a lot to grow in the sense of language models: the most important works done so far are related to the development of these LLMs; how can we actually use this, outside of engineering, to actually generate scientific value? 

Most part of the retinal project lies on the fact that many cities in Brasil actually do not have any ophthalmologists. Actually, if you look at the ophthalmological census, it is possible to see many cities with thousands of people living in, in which there are no ophthalmologists at all. That being said, many doctors in this specific area may find themselves under a lot of pressure to take care of all the pacients. It would be easier for these doctors if they could have a broader idea of what they are observing, and could just validate the hipothesis, instead of needing to deeply analyze every case.

Therefore, our idea arises: we can create a retinal image analysis pipeline, in which, after passing through a screening algorithm to analyze the image quality (coming from our previous research), we could generate a "diagnosis" for what the image is showing. It could say that the eye image represents a health retina, or it could say something about the medical conditions presented. Take the example:

> "[Diabetic Retinopathy]: The image may show diabetic retinopathy, given the dark stain on the top left corner of the retina."

This is just an example, we don't have any data in this format, at least for now. However, our goal is to train a model that can generate a natural language diagnosis from retinal images. This is not meant to replace actual doctors, but to help them look for information in the image, in an easier way. 

## Next Steps

In the next few weeks, we will be defining how we will gather data to train this model and actually choosing an existing model architecture to implement. We may also define a baseline architecture and look for alternatives in the chosen architecture, in a way to increase our results.

- [ ] Understand and study the problem
- [ ] Analyze dataset feasibility
- [ ] Talk to POCs regarding data
    - [ ] Gustavo, from FMUSP, to discuss data annotation
    - [ ] Diogo, to understand the current status of the data we have available

> This post is used as a bachelor thesis checkpoint for the image captioning on retinal fundus images, supervised by professor [Nina S. T. Hirata](https://www.ime.usp.br/~nina/), from the Institute of Mathematics, Statistics and Computer Science at the University of São Paulo (IME USP).
{: .prompt-info }