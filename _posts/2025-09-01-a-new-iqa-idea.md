---
title: "[IQA #8] A new idea for the Focus problem"
date: 2025-09-01 18:30:00 +/-0300
categories: [Undergraduate Research, Retinal Fundus image quality assessment]
description: "This post is just a sketch on the idea I had to solve the Focus problem for IQA in retinal fundus images. I will be still maturing this idea and discussing this with my advisor."
tags: [machine learning, AI, ML, IQA, retina, classification, YOLO, detection]
math: true
---

It has been a little while without posting anything. I have been focusing on finishing my research paper and applying for an internship. I am now happy to share that I am headed to Salvador, to present my research paper on IQA, where I solved the Image Field problem, on SIBGRAPI 2025, while also waiting for the approval of another research paper on the same event, for the Main Track, on Few-Shot Learning for disease classification on retinal images. Also, I am happy to share that I will be joining Uber as a Software Engineer Intern in October, so probably I will have less time to post research updates here.

### A new hope (is this a Star Wars reference?)

Ever since I finished my research project on IQA for the BRSet dataset, where I solved the Image Field parameter, I have been a little unmotivated with research, since I could not have many ideas. This post, I am just talking a little bit about an idea that I had that may be appliable for a similar IQA problem, also present in the BRSet dataset. I know we have not solved the third topic on the BRSet Image Field protocol, but this seems to be hard and, maybe, this new problem would help us figure that one out.

Take a quick look at the BRSet quality assessment table:

![Quality Parameters table]({{ '/assets/img/2025-05-02-iqa-on-retinal-images/quality-parameters-table.png' | relative_url }})
_Quality assessment parameters for BRSet_

This table has been presented here in a prior post, but I want to focus on the _focus_ field. We need to identify the focus for third-generation branches within 1DD around the macula. If you recall my previous research topic, I have already found ways to get the DD measure and locate the center of the macular region, given a retinal image, using YOLO. So, everything we need to do is grade the focus in this region. How can we do that?

Actually, I have no clue if this would work, but a colleague of mine has been working for the past year with segmentation of retinal vessels, and has published a research paper on his methods enhance the segmentation accuracy of thinner vessels. His results are actually impressive, considering the problem, but it is still not quite as good to actually use his model on something like this. However, we may be able to improve his work specifically for this problem. My idea is, use my YOLO models in order to detect the center of the macula, and crop this region within 1DD to each side, we can also crop the segmentation masks and train a U-Net (or improved) on this new cropped images, in order to find the thin vessels only in the region we are interested in.

Looking at this from an outside point of view, this seems doable, but I am not quite sure if the results would be positive. I would also need to think about metrics that could evaluate if our segmentation is good, and make sure this reveals information about the focus. For now, this is just an idea of a problem to work on, since I need to add more things to my course conclusion project.

## Next Steps

For now, there is not much to do, but I will be talking to my advisor and checking if this idea makes sense, and we would have the means to work on top of it.

- [X] Understanding and studying the problem
- [X] Choosing dataset
    - [X] Preprocessing data
- [X] Define training baseline
- [X] Applying YOLO to detect Optic Disc (OD)
    - [X] Obtaining the disc diameter (DD)
    - [X] Obtaining distance from the OD to the Nasal Edge
- [X] Detect Macular Center
    - [X] Obtaining the distance from the Macular Center to the Temporal Edge
- [X] Run statistical analysis on BRSet
    - [X] BRSet image field label distributions
    - [X] OD/Fovea angle distribution
- [X] Paper submission for SIBGRAPI
- [X] Deeper analysis into the original image field protocol
- [ ] Assessing visibility for the inferior and superior arcades


> This post is used as a research checkpoint for the image quality assessment research on retinal fundus images, supervised by professor [Nina S. T. Hirata](https://www.ime.usp.br/~nina/), from the Institute of Mathematics and Statistics at the University of São Paulo (IME USP). The project is supported by [USP/MCTI/Softex/Motorola](https://synestech.ai/).
{: .prompt-info }