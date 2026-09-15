---
title: "[TCC] Training a classifier with CLIP"
categories: [TCC, Multimodal]
date: 2026-09-13 21:00:00 +/-0300
description: "This post covers the process of training a classifier with CLIP, including the training loop and grid-search for parameter tuning."
tags: [machine learning, AI, ML, IQA, retina, LLM, captioning, multimodal, deep learning, DL]
math: true
---

On the [last post](https://rodrigocmichelassi.github.io/posts/zero-shot-classification-clip/) we have covered the process of performing zero-shot classification with CLIP, proposing an implementation fo the evaluation algorithm for the model, and also explored the performance of the model for zero-shot in a context-specific dataset scenario. On this post, we will leverage that algorithm to train CLIP for image classification and make a parameter-tuning approach to have the best model. This post should be a faster read, for any details on the data used for training, check the last post.

Access the [project repository on Github](https://github.com/rodrigocmichelassi/bachelor-thesis).

### Theory Fundamentals -- LoRA

[Low-Rank Adaptation (LoRA)](https://arxiv.org/abs/2106.09685) is a Parameter-Efficient Fine-Tuning (PEFT) tool, that is used to train mainly Large Language Models (LLMs) by consuming only a fraction of the GPU memory that would be used to perform a classical fine-tuning in the model. This is specially important for LLMs because these models are extremely large and an actual bottleneck for the most common scenarios, which don't have enough compute power to perform fine-tuning; however it is widely applied as a fine-tuning method for other models as well.

The way LoRA works is, for each weights matrix available in our model, we freeze the weights $$\mathit{W^{d \times d}}$$ (so we don't perform any gradient updates on the matrix), but we generate two new matrices, $$\mathit{A}^{d \times r}$$  and $$\mathit{B}^{r \times d}$$, in which $$r$$ is the rank, and we perform gradient updates on these matrices. We then proceed to compute the update $$\Delta \mathit{W} = \mathit{A} @ \mathit{B}$$. Given that $$\mathit{A}$$ and $$\mathit{B}$$ have ranks at most $$r$$, then the resulting matrix from the dot product will have rank at most $$r$$ as well. I won't bother going through the proof of that theorem, as I don't really understand it really well myself, but you can find a discussion on this page: [matrix rank inequality](https://math.stackexchange.com/questions/828179/matrices-and-rank-inequality).

Knowing that, for Transformers models, that have several weight matrices, such as the attention weights, in several layers, so we would have the $$\mathit{A}$$ and $$\mathit{B}$$ matrix pair for each of these layers. We leverage this by using hugging face implementation, since it matches what we were doing already to perform zero-shot evaluation on CLIP:

```py
from peft import LoraConfig, get_peft_model

def load_lora_clip(debug=False):
    model = CLIPModel.from_pretrained("openai/clip-vit-base-patch32")
    processor = CLIPProcessor.from_pretrained("openai/clip-vit-base-patch32")

    config = LoraConfig(
        r=8,
        lora_alpha=16, # ΔW = (alpha / r) * A @ B -> usually alpha = 2*r
        target_modules=["q_proj", "v_proj"],  # attention projections, as per the LoRA paper
        lora_dropout=0.1,
        bias="none",
    )

    model = get_peft_model(model, config)
    
    if debug:
        model.print_trainable_parameters()
    
    return model, processor
```

Now pay attention to one slight detail on using LoraConfig: we only target the $$\mathit{Q}$$ and $$\mathit{V}$$ attention weight matrices for LoRA training. The motivation behind this comes from the original LoRA paper, that shows that if we only train the $$\mathit{Q}$$ and $$\mathit{V}$$ attention weight matrices we obtain a really similar performance to training all the weights matrices, as shown in the below figure:

![performance]({{ '/assets/img/2026-09-14-training-clip-classification/performance.png' | relative_url }})
_Rank Performance Table for Transformers with LoRA_

### Training Algorithm

Now that we have understood the fundamentals behind LoRA, we can proceed to fine-tuning CLIP for our classification task. For that, we will be using the same classification dataset described in the previous post, with the model and processor imported in the code snippet from the last section.

For training, we chose to perform a grid-search on the parameters, in order to get the best possible results. We will train each model for 40 epochs, using the AdamW optimizer, searching on learning rate and L2 regularizer value. To best execute the code, we created a bash script:

```sh
#!/bin/bash

export HF_HUB_OFFLINE=1
export TOKENIZERS_PARALLELISM=false

plots_dir="./data/plots"
runs_dir="./logs/runs"

mkdir -p "$plots_dir"
mkdir -p "$runs_dir"

for lr in 1e-3 1e-4 1e-5 1e-6; do
  for l2 in 0.0 0.01 0.001; do
    python main.py --gpu 0 --lr "$lr" --epochs 40 --l2 "$l2" \
      --save_plots_path "${plots_dir}/lr_${lr}_l2_${l2}.png" \
      >> "${runs_dir}/lr_${lr}_l2_${l2}.log" 2>&1
  done
done
```

in which `export HF_HUB_OFFLINE=1` is responsible for using the previous downloaded weights from hugging face, instead of trying to redownload the model everytime (since internet connection was causing a lot of friction) and `export TOKENIZERS_PARALLELISM=false` removes the automatic parallelism from hugging face library, implemented in Rust, to prioritize the usage of PyTorch parallelism, using `num_workers > 0`.

Claude helped us write a function for the contrastive loss (symmetric cross-entropy loss), since it is not automatically part of PyTorch's library. This implementation is also shown in the original CLIP paper:

```py
def contrastive_loss(image_embeds, text_embeds, logit_scale):
    image_embeds = image_embeds / image_embeds.norm(p=2, dim=-1, keepdim=True)
    text_embeds = text_embeds / text_embeds.norm(p=2, dim=-1, keepdim=True)

    logits_per_image = logit_scale * image_embeds @ text_embeds.T
    logits_per_text = logits_per_image.T

    batch_size = image_embeds.shape[0]
    labels = torch.arange(batch_size, device=image_embeds.device)

    loss_i = F.cross_entropy(logits_per_image, labels)
    loss_t = F.cross_entropy(logits_per_text, labels)

    return (loss_i + loss_t) / 2
```

For the training loop, each epoch we iterate through the batches of data, calculate the text and image embeddings, calculate the loss given the embeddings and update the training weights. Notice that hugging face library already does most of the work, such as freezing the attention weight matrices and creating the $$\mathit{A}$$ and $$\mathit{B}$$ matrix-pair. The epoch training is given by:

```py
def train_one_epoch(device, model, optimizer, train_loader):
    model.train()
    running_loss = 0.0

    for batch in train_loader:
        optimizer.zero_grad()

        input_ids = batch["input_ids"].to(device)
        attention_mask = batch["attention_mask"].to(device)
        pixel_values = batch["pixel_values"].to(device)

        text_embeds = model.get_text_features(input_ids=input_ids, attention_mask=attention_mask)
        image_embeds = model.get_image_features(pixel_values=pixel_values)

        logit_scale = model.logit_scale.exp()

        loss = contrastive_loss(image_embeds, text_embeds, logit_scale)
        loss.backward()

        optimizer.step()

        running_loss += loss.item()

    return running_loss / len(train_loader)
```

We proceed to calculate the performance on the validation set (and also the training set), including the accuracy, using the evaluation method presented in the previous post, and use the validation accuracy to save the best model. We also added a few things, such as a plot for accuracy and loss evolution between train and validation sets, so we can see whether the model is overfitting at the end, and early stopping of 7 epochs without improvement to cancel training earlier.

### Results

As for this moment, we have just started running all 12 training loops. We will update this page with the best model results once it's all finished.

Given that, we are now pretty much ready to move on to the next step of this, which is training the model, with the chosen-parameter combination, for the labels used for the image retrieval task! The training process should be the same, but with different data. The evaluation is a different step, instead of returning the caption with the maximum similarity, we want to be using the captions to return the top-K biggest similarity images -- adapting the k-NN algorithm. It will be fun to follow along with these next steps.

I also have a scheduled meeting with my advisor to decide on what will be our priorities for now on this project, given that our timeline is getting tighter.

### Next steps

- [ ] Study the literature for multimodal text-image alignment models.
    - [X] CLIP.
    - [ ] SigLIP.
- [ ] Study the literature to find how these models are used for medical images.
    - [ ] How they are currently leveraged in the literature.
    - [ ] Which metrics are used to evaluate the models performance.
    - [ ] What are the application goals for these models.
- [X] Study the literature for CLIP fine-tuning
    - [X] LoRA
- [X] Adapt dataset for training.
- [X] Analysis of the generated dataset.
- [ ] Fine-tuning and evaluation for CLIP.
    - [X] Zero-Shot Learning Classification
    - [X] Fine-tuning for Classification
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