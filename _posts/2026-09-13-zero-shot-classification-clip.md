---
title: "[TCC] Zero-Shot Classification Using CLIP"
categories: [TCC, Multimodal]
date: 2026-09-13 21:00:00 +/-0300
description: "This post covers our decision and algorithm to perform zero-shot classification using the CLIP model for image-text embeddings alignment."
tags: [machine learning, AI, ML, IQA, retina, LLM, captioning, multimodal, deep learning, DL]
math: true
---

This part of my bachelor thesis was made a few weeks ago. This post comes a bit late since I was focusing on finishing the classification algorithm for this project, in order to move on to the next steps. Given that, the [Next steps](#next-steps) section is marking a lot more points than this post actually covers.

We still need to move forward with other parts of the project, that got really delayed, but now that we have CLIP trained we hopefully will be able to make progress faster.

## Generating Captions for Image Classification

In the [previous post](https://rodrigocmichelassi.github.io/posts/generating-captions/) I went through how I used a local LLM model to generate captions to train the contrastive learning models, given strong signals we had from our datasets. However, as my advisor said, it could be a great idea to train CLIP for classification, as they do in the original paper, that way we can have a sense of whether the performance is according to expected, and how the model actually works. Even though our main goal in this project is not to have the best performance, it is good to have a sense of how things work and the actual performance, since for the image retrieval task we are not able to correctly evaluate the model's performance.

To make this possible, I had generated a set of labels for each image; however, for image classification, CLIP requires less-fancy captions. The code we implemented [is available on github](https://github.com/rodrigocmichelassi/bachelor-thesis/blob/main/scripts/data-generation.py), under the `--class_captions` argument on execution. To make this quicker, for images containing diseases or images of healthy eyes, we created 4 patterns of sentences to represent the images:

```py
DISEASE_TEMPLATES = [
    "This is an image of {disease}.",
    "This retinal image shows {disease}.",
    "A fundus image with signs of {disease}.",
    "Diagnosis: {disease}.",
]

HEALTHY_TEMPLATES = [
    "This is an image of a healthy retina.",
    "This retinal image shows no signs of disease.",
    "A healthy fundus image with no abnormalities.",
    "Diagnosis: healthy retina.",
]
```

we then proceed by sampling a list of three captions for each image and append them to a `.csv` file:

```py
for _, row in brset_df.iterrows():
    image_name = row['image_id'] + '.jpg'
    class_captions, class_name = get_classification_caption_from_row(row)

    captions.append({
        "image_id": image_name,
        "class": class_name,
        "caption_1": class_captions[0],
        "caption_2": class_captions[1],
        "caption_3": class_captions[2]
    })
```

This explores one really important characteristic in CLIP, which let's this kind of model take advantage on a classic classification model: prompt ensembling. Usually, we just have a dataset of images and a mapping saying to which class this image belongs; as a opposite of that, CLIP enables the usage of more details on the sentence that describes the image class. Actually, for models like CLIP, using only the word that describes the class can be even bad, because the same word can have more than a single meaning, and this is visible in ImageNet. Actually, on the CLIP original paper, the authors found that if the sentence gives more context, then the model can perform better on classification tasks.

With this, now we have captions that can be used for zero-shot evaluation of our CLIP model.

### Zero-Shot Evaluation

In a general machine learning, zero-shot means adapting to unseen categories of objects during image classification. In CLIP, however, zero-shot is called by the authors as the study of generalization of entire datasets. In our specific scenario, this refers applying CLIP to a dataset with data it has never seen -- such as BRSET. It is worth flagging that we were not expecting results here to be good (as they were not), since BRSET is a dataset for such a specific task, with medical data that was probably not available in the dataset used to train CLIP.

**Dataset and Model Definition**

To implement CLIP, we use the Hugging Face Transformers library, joined by some coding help from Claude. We start by defining our dataset, that contains the csv for the captions, the directory to find the images and the columns of the csv that contains the captions:

```py
class RetinalClassCaptionDataset(Dataset):
    def __init__(self, captions, image_dir):
        self.captions = captions.reset_index(drop=True)
        self.image_dir = image_dir
        self.caption_cols = ["caption_1", "caption_2", "caption_3"]

    def __len__(self):
        return len(self.captions)

    def __getitem__(self, idx):
        row = self.captions.iloc[idx]

        image_path = f"{self.image_dir}/{row['image_id']}"
        image = Image.open(image_path).convert("RGB")

        # sample a different caption each time this item is fetched
        caption = row[random.choice(self.caption_cols)]
        class_label = row["class"]

        return image, caption, class_label
```

Note that `__getitem__` is a default-needed PyTorch method for datasets, used during training, in which we sample the image and a single caption, out of the three we generated.

We also make a split of our dataset as 70% of the data used for training, 15% for validation and 15% for testing, with a pre-defined seed. We proceed to get the Dataloader by using a Collate Function, that uses the CLIP processor to wrap image and text tensors, as well as the attention mask and class labels.

Our used CLIP model is a ViT base with 32x32 pixel patches as input tokens for the vision encoder, and the text encoder is an usual Transformer.

**CLIP Zero-Shot Classification**

The same method we use to perform the Zero-Shot Classification for CLIP is reused to evaluate CLIP's performance when fine-tuning the model -- with the difference that here we serve the base CLIP model for the task. 

Now notice that we previously defined several image captions for training the classification, however there is a total of 12 diseases + healthy retinas in our dataset, so we want to perform classification across 13 classes. This way, we proceed to generate class captions to which CLIP must predict:

```py
def build_class_labels():
    DISEASE_COLUMNS = [
        'diabetic_retinopathy', 'macular_edema', 'scar', 'nevus', 'amd',
        'vascular_occlusion', 'hypertensive_retinopathy', 'drusens',
        'hemorrhage', 'myopic_fundus', 'increased_cup_disc', 'other_abnormalities'
    ]

    class_labels = {"healthy": "This is an image of a healthy retina."}

    for disease in DISEASE_COLUMNS:
        disease_str = disease.replace('_', ' ')
        class_labels[disease_str] = f"This is an image of {disease_str}."

    return class_labels
```

and we are ready to make the zero-shot evaluation of our dataset. After importing the model from Hugging Face, we follow the pseudo-code:

1. Extract and normalize labels embeddings -- `text_embeds[13, embed_dim]`
2. Iterate through the image batches
3. Extract and normalize the batch image embeddings -- `image_embeds[batch_size, embed_dim]`
4. Calculate the cosine similarity between the embeddings (CLIP Matrix) -- `similarity[batch_size, 13]`
5. For each image, get the label with biggest similarity value (classification label)
6. Calculate metrics after the training loop

The algorithm for the training loop is given:

```py
def evaluate_dataset(model, processor, test_loader, class_labels, device, debug=False):
    # Batch size: 32, len(class_labels): 13
    model.to(device)
    model.eval()

    class_names = list(class_labels.keys())
    prompts = list(class_labels.values())

    text_inputs = processor(text=prompts, return_tensors="pt", padding=True).to(device)

    # Get text embeddings from pre-defined text labels
    # normalize to calculate cosine similarity
    with torch.no_grad():
        text_embeds = model.get_text_features(**text_inputs)
        text_embeds = text_embeds / text_embeds.norm(p=2, dim=-1, keepdim=True) # text_embeds: [13, embed_dim]

    if debug:
        text_sim = text_embeds @ text_embeds.T  # [13, 13]
        print(text_sim) 

    true_labels = []
    predicted_labels = []

    running_loss = 0.0
    with torch.no_grad():
        for batch in test_loader:

            pixel_values = batch["pixel_values"].to(device)
            batch_true_labels = batch["class_labels"]

            image_embeds = model.get_image_features(pixel_values=pixel_values)
            image_embeds = image_embeds / image_embeds.norm(p=2, dim=-1, keepdim=True)  # image_embeds: [batch_size, embed_dim]

            logit_scale = model.logit_scale.exp()
            # calculates clip similarity matrix
            similarity = logit_scale * image_embeds @ text_embeds.T  # cosine similarity, [batch_size, 13], CLIP matrix

            # classification loss: cross-entropy against the true class index
            true_idx = torch.tensor(
                [class_names.index(label) for label in batch_true_labels],
                device=device
            )
            loss = F.cross_entropy(similarity, true_idx)
            running_loss += loss.item()

            # for each row (batch image), gets the max score for cosine similarity
            # (classification label for each image)
            batch_predicted_idx = similarity.argmax(dim=1)
            batch_predicted_labels = [class_names[i] for i in batch_predicted_idx]

            true_labels.extend(batch_true_labels)
            predicted_labels.extend(batch_predicted_labels)

    loss = running_loss / len(test_loader)

    return true_labels, predicted_labels, loss
```

Hopefully this gives an overview of how CLIP evaluation is done. This is all described in the section "3.1.2. USING CLIP FOR ZERO-SHOT TRANSFER" from the CLIP original paper.

**Results**

To evaluate the results obtained, we put our efforts into calculating the accuracy, the confusion matrix and the classification report from Sklearn. We also calculate the Balanced Accuracy, which is the accuracy calculated over each class, given that our dataset is fairly imbalanced.

The results follow:

```md
Accuracy: 0.009197751660705161
Balanced Accuracy: 0.08880090497737556
Classification Report:
                          precision    recall  f1-score   support

                 healthy       0.00      0.00      0.00      1199
    diabetic retinopathy       0.10      0.03      0.04        68
           macular edema       0.00      0.00      0.00         2
                    scar       0.00      0.00      0.00        12
                   nevus       0.00      0.00      0.00         9
                     amd       0.00      0.00      0.00         7
      vascular occlusion       0.00      0.00      0.00         3
hypertensive retinopathy       0.01      1.00      0.02        15
                 drusens       0.00      0.00      0.00       255
              hemorrhage       0.00      0.00      0.00         4
           myopic fundus       0.01      0.12      0.01         8
      increased cup disc       0.00      0.00      0.00       319
     other abnormalities       0.00      0.00      0.00        56

                accuracy                           0.01      1957
               macro avg       0.01      0.09      0.01      1957
            weighted avg       0.00      0.01      0.00      1957

Confusion Matrix:
[[   0   10    1    0    0    0    0 1079    0    0  109    0    0]
 [   0    2    0    0    0    0    0   62    0    0    4    0    0]
 [   0    0    0    0    0    0    0    2    0    0    0    0    0]
 [   0    0    0    0    0    0    0   10    0    0    2    0    0]
 [   0    0    0    0    0    0    0    8    0    0    1    0    0]
 [   0    2    0    0    0    0    0    4    0    0    1    0    0]
 [   0    0    0    0    0    0    0    3    0    0    0    0    0]
 [   0    0    0    0    0    0    0   15    0    0    0    0    0]
 [   0    5    0    0    0    0    0  221    0    0   29    0    0]
 [   0    1    0    0    0    0    0    3    0    0    0    0    0]
 [   0    0    0    0    0    0    0    7    0    0    1    0    0]
 [   0    1    0    0    0    0    0  299    0    0   19    0    0]
 [   0    0    0    0    0    0    0   52    0    0    4    0    0]]
```

As it is possible to see, the results are far far from reality. Actually, most of the predictions went for a single class: hypertensive retinopathy. Even though we expected bad results, this is not something we'd expect. 

In order to investigate this, Claude suggested we'd calculate the similarity matrix between the text embeddings and themselves: `text_sim = text_embeds @ text_embeds.T`, given that the text labels are way too similar for all the images (a simple "This is an image of {disease_str}."). That shows that, if the similarity between classes is very high, then probably the issue is just that all the labels are really similar -- and it was. That way, CLIP text encoders could not really separate the text embeddings really well.

Notably, when doing that, the hypertensive retinopathy showed a really high similarity index with every other class available. 

That way, zero-shot classification with CLIP was proven not worth for really specific classification tasks, in niche-specific datasets. However, it was an important step to figure out how CLIP prediction actually works. We were missing only fine-tuning the CLIP model to get a more detailed results. To do this next step, we will be using Low-Rank Adaptation (LoRA), a Parameter-Efficient Fine-Tuning method used mainly for large language models (not our case) which allows us to train much smaller weight matrices at the same time as it is a nice opportunity for learning how something new works :)

The next post will cover fine-tuning CLIP with LoRA

### Next steps

- [ ] Study the literature for multimodal text-image alignment models.
    - [X] CLIP.
    - [ ] SigLIP.
- [ ] Study the literature to find how these models are used for medical images.
    - [ ] How they are currently leveraged in the literature.
    - [ ] Which metrics are used to evaluate the models performance.
    - [ ] What are the application goals for these models.
- [ ] Study the literature for CLIP fine-tuning
    - [ ] LoRA
- [X] Adapt dataset for training.
- [X] Analysis of the generated dataset.
- [ ] Fine-tuning and evaluation for CLIP.
    - [X] Zero-Shot Learning Classification
    - [ ] Fine-tuning for Classification
    - [ ] Image search by natural language
- [ ] Training and evaluation for SigLIP.
- [ ] Results analysis and comparison.
- [ ] Code organization for the model training.
    - [ ] Pre-defined seeds for traning and results reproduction.
    - [X] Scalable architecture for ML models training systems.
- [ ] Develop tool for search in images databases.
- [ ] Make all source code available as an open-source project.
- [ ] Write thesis (WiP)
- [ ] ~~(Bonus): follow the same development pattern to train a VLM model capable of generating captions for retinal images~~

> This post is used as a checkpoint for my bachelor thesis project: retinal image retrieval pipeline via natural language, supervised by professor [Nina S. T. Hirata](https://www.ime.usp.br/~nina/), from the Institute of Mathematics, Statistics and Computer Science at the University of São Paulo (IME USP).
{: .prompt-info }