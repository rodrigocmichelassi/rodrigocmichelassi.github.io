---
title: "[IQA EXTRA #1] Is BRSet mislabeled?"
date: 2025-05-02 17:00:00 +/-0300
categories: [Undergraduate Research, Retinal Fundus image quality assessment]
description: "This post follows an observation made after IQA #2 post, based on the usage of the model we trained on a specific image."
tags: [machine learning, AI, ML, IQA, retina, classification, YOLO, detection]
---

From what we have done so far, training the object detection model is possibly the most important thing in order to solve the Image Field parameter to assess quality to a retinal image. After training the model, it is useful to test it on some images, also to know if our model and the results are working as expected.

For that, looking for a nice looking image on BRSet to test, we came across `img10829.jpg`, that is labeled as Adequate. If we remember the first post for the IQA research project, a image is only labeled as Adequate if it does not present any quality problems, according to the parameters definition. With that in mind, we ran our model and the algorithms written to check if this image would pass the first Image Field criteria: "1. The optic disc is at least 1 disc diameter (DD) from the nasal edge". The image follows: 

![Quality Parameters](/assets/img/2025-05-02-iqa-on-retinal-images/quality-parameters.png)
_Retinal image presenting labeling problem_

If you remember the description given, the nasal edge is the retinal edge closest to the optic disc, in this image case, the right side. Even visually, it is possible to see that the distance from the optic disc to the nasal edge is obviously smaller than 1DD, but, to be exact, on our algorithm, we got that this distance is equivalent to 218.54 pixels, while 1DD is equivalent to 451.09 pixels. 

Trying to think outside the box for this problem, I have also calculated the distance from the center of the optic disc to the nasal edge, obtaining a distance of 444.09 pixels, less than 10 pixels smaller than 1DD. That led me to think that I could stablish the distance between the optic disc center and the nasal edge, and maybe attribute an interval to accept errors on the distance, based on the 1DD, given that the images were probably annotated by doctors, who could make mistakes. Talking to Dr. Luis Nakayama, he stated that the distance should be measured based on the corner of the optic disc, not its center, so I did not do that.

By facing this problem, my advisor said that, a nice analysis we could be doing is applying the final model (after everything is done) to the whole BRSet inadequate images, and make a statistical analysis, concerning how the dataset may be adjusted, and how mistakes may be present when doing projects like this only by eye.

## Next Steps

- [X] Understanding and studying the problem
- [X] Choosing dataset
    - [X] Preprocessing data
- [X] Define training baseline
- [X] Applying YOLO to detect Optic Disc (OD)
    - [X] Obtaining the disc diameter (DD)
    - [X] Obtaining distance from the OD to the Nasal Edge
- [ ] Detect Macular Center
    - [ ] Obtaining the distance from the Macular Center to the Temporal Edge
- [ ] Assessing visibility for the inferior and superior arcades


> This post is used as a research checkpoint for the image quality assessment research on retinal fundus images, supervised by professor [Nina S. T. Hirata](https://www.ime.usp.br/~nina/), from the Institute of Mathematics and Statistics at the University of São Paulo (IME USP). The project is supported by [USP/MCTI/Softex/Motorola](https://synestech.ai/).
{: .prompt-info }