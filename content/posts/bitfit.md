+++
title = 'BitFit: BIas-Term FIne-Tuning'
date = 2025-02-19T15:20:48+01:00
draft = false
tags = ["NLP", "Machine Learning", "Fine-tuning", "BitFit", "BERT"]
+++

<style
TYPE="text/css">

code.has-jax {font:
inherit;
font-size:
100%; 
background: 
inherit; 
border: 
inherit;}

</style>

<script
type="text/x-mathjax-config">

MathJax.Hub.Config({

    tex2jax: {

        inlineMath: [['$','$'], ['\\(','\\)']],

        skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'] // removed 'code' entry

    }

});

MathJax.Hub.Queue(function() {

    var all = MathJax.Hub.getAllJax(), i;

    for(i = 0; i < all.length; i += 1) {

        all[i].SourceElement().parentNode.className += ' has-jax';

    }

});

</script>

<script
type="text/javascript"
src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.4/MathJax.js?config=TeX-AMS_HTML-full"></script>

<h1 style="font-size: 24px;">BitFit: A Simpler and More Efficient Approach to Fine-tuning Transformers</h1>

### Authors : Abdoul R. Zeba, Nour Yahya, Nourelhouda Klich

<h2 style="font-size: 20px;"> 1. Introduction </h2>

Fine-tuning large transformer models like BERT has become the gold standard for adapting them to specific tasks. However, this process is often computationally expensive, requiring vast amounts of memory, making it impractical for many real-world applications. What if there was a way to adapt these models with minimal computational overhead while maintaining competitive performance?  

Through this blog post, we will discuss <i>**BitFit**</i> —  a novel parameter-efficient fine-tuning technique proposed in the paper BitFit: A Simpler and More Efficient Approach to Fine-tuning Transformers ([Ben Zaken et al., 2022](#benzaken)).  


<h2 style="font-size: 20px;"> 2. Why Fine-tuning Needs Optimization </h2>

Fine-tuning NLP models typically involves updating all model parameters, but this poses some major challenges:  
1. Computational Cost: Training a full BERT-large model requires high-end GPUs and significant memory.  
2. Deployment Complexity: Every task-specific fine-tuned model requires a separate copy of the large model.  
3. Transfer Learning Issues: Modifying too many parameters can lead to overfitting on small datasets.  

Wouldn't it be great if we could fine-tune just a small subset of the parameters and get similar results?  This is exactly where <i>**BitFit**</i> comes in.  

<h2 style="font-size: 20px;"> 3. How BitFit Works  </h2>

Traditional fine-tuning updates **all** the parameters in a Transformer model, which is computationally expensive. BitFit, on the other hand, **only updates the bias terms** in the model while keeping all other weights **frozen**.

> **Bias terms** are small but essential parameters in neural networks. They adjust activations before applying transformations, helping models adapt to new tasks with minimal updates.

<h3 style="font-size: 18px;"> 3.1 Why Focus on Bias Terms ? </h3>

Bias terms **<i>b</i>** play a crucial role in neural networks because:
- They allow neurons to fire (activate) even when inputs are zero.
- Adjusting bias values can shift outputs without requiring full retraining.
- Updating biases is a lightweight operation, meaning less memory and faster adaptation.

A key finding in the BitFit paper is that bias terms contribute uniquely to fine-tuning. When researchers randomly selected the **same number** of non-bias parameters for fine-tuning, the model performed significantly worse than BitFit &rarr; This suggests that bias parameters are not just a small subset but play an important role in model adaptation.  

<h3 style="font-size: 18px;"> 3.2 What Layers Does BitFit Modify ? </h3>

BitFit updates the bias terms in **key layers** of Transformer models like BERT:

- **Self-Attention Layers**  

Transformers use self-attention to focus on important words in a sentence. Each attention head contains three transformations: **Query** (`Q`), **Key** (`K`), **Value** (`V`). And each transformation has its own bias term (`bQ`, `bK`, `bV`).  

&rarr; BitFit **updates only these biases** (`bQ`, `bK`, `bV`), while the main attention weights (`WQ`, `WK`, `WV`) remain **unchanged**.

- **Feedforward Layers (MLPs)**  

Transformers contain fully connected layers that transform intermediate representations. These layers consist of two main **weight matrices** (`W1`, `W2`) and **bias terms** (`b1`, `b2`).

&rarr; BitFit **updates only the bias terms** (`b1`, `b2`), keeping the weight matrices (`W1`, `W2`) **frozen**.

- **Layer Normalization (LN) Layers**  

Transformers use Layer Normalization to stabilize training and prevent exploding gradients. Each LN layer has two learnable parameters: the **scale factor** `$\gamma$` to control the output scaling and the **bias** `$\beta$` to adjust the mean shift.

&rarr; BitFit **modifies only the bias term** `$\beta$`, while keeping the scale factor `$\gamma$` **fixed**.

<h3 style="font-size: 18px;"> 3.3 Mathematical Explanation </h3>
<h4 style="font-size: 16px;"> 3.3.1  Standard Fine-Tuning </h4>

In traditional fine-tuning, we update **both** the weights ($W$) and bias terms ($b$):

$$
W_{\text{fine-tuned}} = W_{\text{pretrained}} + \Delta W
$$

$$
b_{\text{fine-tuned}} = b_{\text{pretrained}} + \Delta b
$$

where:
- $W_{\text{pretrained}}$ is the original weight matrix from the pre-trained model.
- $\Delta W$ is the learned update during fine-tuning.
- $b_{\text{pretrained}}$ is the original bias vector.
- $\Delta b$ is the learned bias update.

<h4 style="font-size: 16px;"> 3.3.2  BitFit: BIas-Term FIne-Tuning </h4>

Instead of updating all weights, BitFit **freezes** $W$ and **only updates** $b$:

$$
W_{\text{fine-tuned}} = W_{\text{pretrained}} 
$$

$$
b_{\text{fine-tuned}} = b_{\text{pretrained}} + \Delta b
$$

Here, $\Delta b$ represents the learned adjustments needed for the new task.

<h3 style="font-size: 18px;"> 3.4 Are all Bias Terms Equal ? </h3>

Not all bias parameters contribute equally to fine-tuning. Researchers found that two types of bias terms are especially important:  
- **Query Biases** `bQ`: Found in self-attention layers, responsible for selecting relevant words.  
- **Middle-Layer MLP Biases** `b2`: Found in feedforward layers, responsible for transforming representations.  

By only fine-tuning these two subsets, performance remained almost identical to full BitFit while updating half the parameters.  

> BitFit typically fine-tunes **0.08%** of model parameters, but using only `bQ` and `b2`, this number drops to **0.04%** with no major accuracy loss!  

This means fine-tuning can be made even more efficient by selecting only the most impactful bias terms.

<h2 style="font-size: 20px;"> 4. How Well Does BitFit Perform? </h2>

Compared to other parameter-efficient fine-tuning techniques such as Diff-Pruning and Adapters, BitFit achieves competitive performance with significantly fewer trainable parameters.

BitFit outperforms Diff-Pruning on 4 of the 9 tasks of the GLUE benchmark using the BERTLARGE model and with 6 times fewer trainable parameters. On the test set, BitFit decisively beats Diff-Pruning over two tasks and Adapters over four tasks with 45 times fewer trainable parameters.

The performance trends of BitFit remain consistent across different base models, e.g., BERTBASE and RoBERTaBASE. The performance of BitFit is not simply due to its adaptation of a collection of parameters, but rather the specific choice of bias parameters. Random selection of an identical number of parameters yields significantly poorer performance, which means that bias parameters have a unique critical contribution to fine-tuning.
Moreover, further analysis reveals that not all bias parameters are equally important as some of them contribute more to the model's performance than others.

BitFit also demonstrates a smaller generalization gap compared to full fine-tuning, suggesting better generalization capabilities. In token-level tasks such as POS-tagging, BitFit achieves comparable results to full fine-tuning.

Finally, BitFit's performance also appears to rely on training set size. In experiment with the Stanford Question Answering Dataset, BitFit outperforms full fine-tuning in small-data regimes, but the trend reverses as the training set size increases. What that means is that BitFit is particularly useful when it comes to targeted fine-tuning under small-to-mid-sized data conditions.

<h2 style="font-size: 20px;"> 5. Why Does BitFit Work? </h2>

BitFit's success can be attributed to several key factors that challenge traditional assumptions about fine-tuning large language models. Rather than retraining all parameters, BitFit selectively updates only the bias terms, leading to efficient adaptation without sacrificing performance. But why is this approach effective?

<h3 style="font-size: 18px;"> 5.1 Fine-Tuning as Knowledge Exposure, Not Learning </h3>

A crucial insight is that fine-tuning large pre-trained transformers is often less about "learning new knowledge" and more about "exposing" the knowledge already embedded in the model. Since transformer-based models like BERT have already learned a vast range of linguistic patterns during their unsupervised pre-training phase, adjusting a small number of parameters—specifically the bias terms—can be enough to bring out task-specific information without reworking the entire model.

<h3 style="font-size: 18px;"> 5.2 Bias Terms and Their Unique Role </h3>

Bias terms in neural networks serve as offset values, allowing neurons to activate even when input features are zero. Unlike weights, which define relationships between features, bias terms shift outputs in a task-specific manner.

BitFit leverages the fact that bias terms interact across layers in a way that can subtly adjust how information flows through the model without needing to modify the main weight matrices. This enables significant changes in task-specific performance with minimal modifications to the model structure.

<h3 style="font-size: 18px;"> 5.3 A Targeted and Structured Approach </h3>

Not all bias parameters contribute equally to model adaptation. The study found that:

- **Query Biases (bQ)**: Found in self-attention layers, crucial for determining which words receive attention.

- **Middle-Layer MLP Biases (b2)**: Found in feedforward layers, responsible for transforming hidden representations.

By focusing updates on these specific biases, BitFit achieves near full fine-tuning performance while modifying only 0.04% of model parameters.

<h3 style="font-size: 18px;"> 5.4 The Generalization Advantage </h3>

Another reason BitFit works well is its effect on generalization. Traditional fine-tuning tends to overfit on small datasets because it updates many parameters, potentially memorizing noise rather than learning transferable patterns. BitFit, by contrast, updates only a fraction of the parameters, leading to a smaller generalization gap.

In fact, experiments show that BitFit performs better than full fine-tuning in small-data regimes. This suggests that limiting parameter updates can sometimes lead to better robustness and generalization.

<h2 style="font-size: 20px;"> 6. Implications and Future Directions </h2>

BitFit opens new avenues for efficient fine-tuning, making it particularly relevant in scenarios where computational resources are limited, such as:

<h3 style="font-size: 18px;"> 6.1 Efficient Deployment in Real-World Applications </h3>

- **Low-Resource AI Systems**: BitFit’s lightweight approach is ideal for deploying NLP models in mobile applications, IoT devices, and embedded AI systems where computational efficiency is critical.

- **Multi-Task Learning**: Since only bias terms need updating, multiple tasks can share the same base model, reducing memory overhead and increasing flexibility in production systems.

- **Scalable NLP Services**: Cloud-based NLP services that handle multiple tasks (e.g., chatbots, automated translations) can benefit from BitFit by reducing the need to store and load separate models for each task.

<h3 style="font-size: 18px;"> 6.2 Understanding Model Adaptation </h3>

The success of BitFit raises deeper questions about the nature of transfer learning and fine-tuning:

- Do large transformers truly need full fine-tuning, or is most of their knowledge already latent?

- Could bias-only tuning be the key to unlocking efficient continual learning strategies?

- Are there other small but critical subsets of parameters that can be updated to achieve similar efficiency gains?

<h3 style="font-size: 18px;"> 6.3 Potential Enhancements </h3>

While BitFit is a promising step forward, future research could explore:

- **Selective bias tuning**: Further analyzing which specific bias terms contribute most to adaptation and whether additional optimization can reduce the number of updates even further.

- **Hybrid approaches**: Combining BitFit with methods like adapters or LoRA (Low-Rank Adaptation) to achieve even better efficiency.

- **Application to other architectures**: Investigating whether BitFit’s principles extend beyond BERT to models like GPT, T5, and multimodal transformers.

<h2 style="font-size: 20px;"> 7. Conclusion </h2>

In conclusion, BitFit offers a desirable compromise between effectiveness and efficiency, making it a valuable tool for fine-tuning transformer-based models, especially in resource-constrained environments or with limited amounts of training data. Having the capability to achieve competitive performance using significantly fewer trainable parameters, coupled with its achievement in low data regimes, bodes well for other NLP tasks and applications.

BitFit defies the usual wisdom concerning universal fine-tuning by illustrating how slight tweaks in only a very small percentage of model parameters yield high-performance. This efficient approach makes AI models more accessible, scalable, and cost-effective.


### References  

- <a id="#benzaken"></a> [BitFit: Simple Parameter-efficient Fine-tuning for Transformer-based Masked Language-models](https://aclanthology.org/2022.acl-short.1/) (Ben Zaken et al., ACL 2022)
