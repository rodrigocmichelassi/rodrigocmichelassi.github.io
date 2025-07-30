---
title: "[IQA #7] Image Field definition in fundus images"
date: 2025-07-30 14:00:00 +/-0300
categories: [Undergraduate Research, Retinal Fundus image quality assessment]
description: "In this post, we will dive deeper into the image field definition protocol for fundus images."
tags: [machine learning, AI, ML, IQA, retina, classification, YOLO, detection]
math: true
---

For the past few months, we have been solving the image field definition protocol defined in the BRSet paper. By having some discussions with one of the doctors responsible for the dataset, he stated that this protocol is internationally known and used, so, to write the paper for the project, we thought we should find where this protocol was defined and cite as a reference. In this paper, we will be covering what we found on this.

### The image field protocol

We could not really find where this protocol was first defined, so we turned to language models that could search the internet to find information. GPT suggested that this protocol was defined in the [Automated Assessment of Diabetic Retinal Image Quality Based on Clarity and Field Definition](https://iovs.arvojournals.org/article.aspx?articleid=2183453) paper, published in 2006. I am not completely sure about this information, but it may be true, since the information available in this article is very similar to what we have been working so far.

Take a closer look at the image field definition table:

![image_field_distribution]({{ '/assets/img/2025-07-30-image-field/table.png' | relative_url }})
_Image field protocol table_

If you recall the definition we presented in the [first post of this series](https://rodrigocmichelassi.github.io/assets/img/2025-05-02-iqa-on-retinal-images/quality-parameters-table.png), it is noticeable that some values are different. For instance, for BRSet the angle between the optic disc and the foveal region is not taken into consideration, and the distances values are also different. For instance, in this original paper, the distance between the optic disc and the nasal edge should be greater than 0.5DD, beginning in the optic disc center, while in BRSet it should be greater than 1DD, beginning on the optic disc edge (information obtained by the BRSet doctor we have been in touch). 

Taking this into consideration, the distances analysis made in a [previous post](https://rodrigocmichelassi.github.io/posts/brset-statistical-analysis/) would also change, and probably the mistakes made in the labeling process would be much smaller. On the other hand, we can not be sure if the doctors that labeled BRSet used the original protocol and made a mistake in the table given in the paper, or if they, indeed, used the protocol they presented in the BRSet paper and made mistakes during the annotation process. For our paper, we have decided not to cite this original article, since we did not have enough information about this, and there is just not enough time to get in touch with the doctors and fix this, so it can be a future work.

Looking further into this original paper though, we can find the following image:

![retina_distances]({{ '/assets/img/2025-07-30-image-field/retina.png' | relative_url }})
_Retinal distances calculation_

The authors state that, the distances calculated are, in fact, the shortest distance from the reference points they have defined to the retinal edges. I believe that, when using the geometrical approach, given by the unit vector $$\vec{U}$$ mentioned in the [distances calculation post](https://rodrigocmichelassi.github.io/posts/distance-to-temporal-and-nasal-edges/), we take care of this problem, but now our approach seems to be way more plausible.

### Paper submission to event

Given the analysis just made, I am happy to inform that we have submitted a paper concerning this work to SIBRAPI Workshop of Undergraduate Works, that will take place in Salvador - BA. Our research still leaves gaps for lots of experiments and analysis, but so far we had real world contributions, especially concerning the BRSet dataset analysis and object detection with YOLO for anatomical retinal structures. If this paper gets accepted, I will be updating this post to share the news :)

## Next Steps

With all this ride along the image quality assessment and object detection for retinal images, our work is close to an end. I'm really excited for everything that is to come and how this project is going to grow. For future steps, I am not sure how much work we will be performing on this problem, but I am excited to take biggest steps into deep learning and see how I can use new tools to contribute to the scientific community.

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
- [ ] Deeper analysis into the original image field protocol
- [ ] Assessing visibility for the inferior and superior arcades


> This post is used as a research checkpoint for the image quality assessment research on retinal fundus images, supervised by professor [Nina S. T. Hirata](https://www.ime.usp.br/~nina/), from the Institute of Mathematics and Statistics at the University of São Paulo (IME USP). The project is supported by [USP/MCTI/Softex/Motorola](https://synestech.ai/).
{: .prompt-info }