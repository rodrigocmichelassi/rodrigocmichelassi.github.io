---
title: "[TCC] Defining the theme"
date: 2026-04-06 21:00:00 +/-0300
categories: [TCC, Image captioning]
description: "This post covers a discussion I had with my advisor, to talk about the observations I made on the dataset and what problem we are attacking for the thesis."
tags: [machine learning, AI, ML, IQA, retina, LLM, captioning]
math: true
---

In [the last post](https://rodrigocmichelassi.github.io/posts/initial-steps/) I mentioned a few models I had researched to apply to my thesis project, and mentioned a couple step backs in using the dataset we had available from FMUSP. As of today, after a meeting with my advisor, we decided to postpone the image captioning using VLMs project to later, and maybe attack this after we have an actual idea about how multimodal models works. In this post, I will be covering the decision towards what project we are actually applying to my thesis.

### Taking a step back

By talking to my advisor, we noticed that doing this project maybe was not possible, since we did not have enough interesting images annotations for image captioning in our project. However, that did not mean that we couldn't keep pursuing working towards that direction, but we should probably guarantee we can make some project work.

For anyone who read the posts about my previous undergraduate research project, knows that I have trained a YOLO model to detect optic disc and fovea, which is a very useful information in retinal images. From that not only we can find several diseases in the pacient but we can infer if the image is adequate or not to find helpful information about diabetic retinopathy, which eye does the photo refer to, and other details. Between the algorithms that I have developed, one of them tells the side of the retina that the optic disc is located.

Knowing that information, we just thought, maybe we can consider training a multimodal model, as basic as CLIP, that will allow us to make searches in an image database based on natural language queries. Our idea here is that we can create a few filters, such as "Images with diabetic retinopathy, from the right eye", and then we would obtain exactly those images. We could actually expand this to be able to have many more information that the user can query from the database, regarding retinal images. If everything goes according to plan, we can reuse the image encoder trained in this project to finetune another model, that would then generate image captioning.

However, how do we have access to data that would be possible to extract query informations in natural language? In the era of AI Agents, that should not be an issue! We have previously used the BRSet dataset, that carries a lot of information for over 16.000 images. If we can combine our previous optic disc location algorithm to some information coming from the dataset, we can create a LLM prompt to generate a specific language label for each image. That way, we would have a natural language dataset with descriptions for each image.

### Summing up our goal

Now that we have discussed the motivation, here is our long-term goal for this thesis:

- Use our YOLO algorithm to extract information about the retinal images
- Generate images descriptions with data from BRSet
- Train a CLIP model to align images and text description
- Train a SigLip model to align images and text description
- Compare both models results (numerically)
- Create a database search engine, that would return images based on a prompt
- Engineer towards the database search engine for optimization
- Evaluate the feasibility of VLMs to complete the assignment

We should not be worried about taking a VLM project now, this will be a bonus in case our first idea works as expected. However, we now have a more clear vision of how we will get the data. Not only that, but we can explore data from other domains, such as the BRSet Mobile dataset.

## Next Steps

- [ ] Explore BRSET dataset
    - [ ] Read about BRSET mobile and analyze if it is usable for our project
- [ ] Review CLIP
- [ ] Study SIGLIP
- [ ] Prepare the dataset for training the model
    - [ ] Gather data regarding our YOLO algorithm for each image of the dataset
    - [ ] Integrate LLM to generate captions for the images
- [ ] Train CLIP model
- [ ] Train SIGLIP model
- [ ] Create database search engine
    - [ ] Engineering process on the database (prompt filtering via LLM, to avoid returns from queries not related to retinal images)
- [ ] Start planning the VLM project

> This post is used as a bachelor thesis checkpoint for the image captioning on retinal fundus images, supervised by professor [Nina S. T. Hirata](https://www.ime.usp.br/~nina/), from the Institute of Mathematics, Statistics and Computer Science at the University of São Paulo (IME USP).
{: .prompt-info }