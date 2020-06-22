# GAN-BERT

Code for the paper **GAN-BERT: Generative Adversarial Learning for Robust Text Classification with a Bunch of Labeled Examples** accepted for publication at **ACL 2020 - short papers** by *Danilo Croce* (Tor Vergata, University of Rome), *Giuseppe Castellucci* (Amazon) and *Roberto Basili* (Tor Vergata, University of Rome). The paper can be found [here](https://www.aclweb.org/anthology/2020.acl-main.191.pdf).

GAN-BERT is an extension of BERT which uses a Generative Adversial setting to implement an effective semi-supervised learning schema. It allows training BERT with datasets composed of a limited amount of labeled examples and larger subsets of unlabeled material. 
GAN-BERT can be used in sequence classification tasks (also involings text pairs). 

This code runs the GAN-BERT experiment over the TREC dataset for the fine-grained Question Classification task. We provide in this package the code as well as the data for running an experiment by using 2% of the labeled material (109 examples) and 5343 unlabeled examples.
The test set is composed of 500 annotated examples.

As a result, BERT trained over 109 examples (in a classification task involving 50 classes) achieves an accuracy of ~13% while GAN-BERT achieves an accuracy of ~42%.

## The GAN-BERT Model

GAN-BERT is an extension of the BERT model within the Generative Adversarial Network (GAN) framework (Goodfellow et al, 2014). In particular, the Semi-Supervised GAN (Salimans et al, 2016) is used to make the BERT fine-tuning robust in such training scenarios where obtaining annotated material is problematic. In fact, when fine-tuned with very few labeled examples the BERT model is not able to provide sufficient performances. With GAN-BERT we extend the fine-tuning stage by introducing a Discriminator-Generator setting, where:

- the Generator G is devoted to produce "fake" vector representations of sentences;
- the Discrimator D is a BERT-based classifier over k+1 categories.

![GAN-BERT model](https://github.com/crux82/ganbert/raw/master/ganbert.jpg)

D has the role of classifying an example with respect to the k categories of the task of interest, and it should recognize the examples that are generated by G (the k+1 category). 
G, instead, must produce representations as much similar as possible to the ones produced by the model for the "real" examples. G is penalized when D correctly classify an example as fake.

In this context, the model is trained on both labeled and unlabeled examples. The labeled examples contributes in the computation of the loss function with respect to the task k categories. The unlabeled examples contributes in the computation of the loss functions as they should not be incorrectly classified as beloning to k+1 category (i.e., the fake category).

The resulting model is demonstrated to learn text classification tasks starting from very few labeled examples (50-60 examples) and to outperform the classifcal BERT fine-tuned models by large margin in this setting.

In the following plots, the performances of GAN-BERT are reported for different tasks at different percentage of labeled examples. We measured the accuracy (or F1) of the model for the following tasks: Topic Classification on the 20News (20N) dataset; Question Classification (QC) on the TREC dataset; Sentiment Analysis on the SST dataset (SST-5); Natural Language Inference over the MNLI dataset (MNLI).

![Performances](https://github.com/crux82/ganbert/raw/master/ganbert_performances.png)

## Requirements

The code is a modification of the original Tensorflow code for BERT (https://github.com/google-research/bert).
It has been tested with Tensorflow 1.14 over a single Nvidia V100 GPU. The code should be compatible with TPUs, but it has not been tested on such architecture or on multiple GPUs.
Moreover, it uses tf_metrics (https://github.com/guillaumegenthial/tf_metrics) to compute some performance measure.

## Installation Instructions
It is suggested to use a python 3.6 environment to run the experiment.
If you're using conda, create a new environment with:

```
conda create --name ganbert python=3.6
```

Activate the newly create environment with:

```
conda activate ganbert
```

And install the required packages by:

```
pip install -r requirements.txt
```

This should install both Tensorflow and tf_metrics.

## How to run an experiment

The run_experiment.sh script contains the necessary steps to run an experiment with both BERT and GANBERT.

The script can be launched with:

```
sh run_experiment.sh
```

The script will first download the BERT-base model, and then it will run the experiments both with GANBERT and with BERT.

After some time (on a Nvidia Tesla V100 it takes about 5 minutes) there will be two files in output: *qc-fine_statistics_BERT0.02.txt* and *qc-fine_statistics_GANBERT0.02.txt*. These two contain the performance measures of BERT and GANBERT, respectively. 

After training a traditional BERT and GAN-BERT on only 109 labeled examples in a classification task involving 50 classes, the following results are obtained:

**BERT**
<pre>
eval_accuracy = 0.136 
eval_f1_macro = 0.010410878 
eval_f1_micro = 0.136 
eval_loss = 3.7638452 
eval_precision = 0.136 
eval_recall = 0.136 
</pre>

**GAN-BERT**
<pre>
eval_accuracy = 0.418 
eval_f1_macro = 0.056867357 
eval_f1_micro = 0.418
eval_loss = 2.744603 
eval_precision = 0.418
eval_recall = 0.418
</pre>


## Out-of-memory issues

As the code is based on the original BERT Tensorflow code and that it starts from the BERT-base model, the same batch size and sequence length restrictions apply here based on the GPU that is used to run an experiment.

Please, refer to the BERT github page (https://github.com/google-research/bert#out-of-memory-issues) to find the suggested batch size and sequence length given the amount of GPU memory available.

## Citation

To cite the paper, please use the following:

<pre>
@inproceedings{croce-etal-2020-gan,
    title = "{GAN}-{BERT}: Generative Adversarial Learning for Robust Text Classification with a Bunch of Labeled Examples",
    author = "Croce, Danilo  and
      Castellucci, Giuseppe  and
      Basili, Roberto",
    booktitle = "Proceedings of the 58th Annual Meeting of the Association for Computational Linguistics",
    month = jul,
    year = "2020",
    address = "Online",
    publisher = "Association for Computational Linguistics",
    url = "https://www.aclweb.org/anthology/2020.acl-main.191",
    pages = "2114--2119"
}
</pre>


## References
- Ian Goodfellow, Jean Pouget-Abadie, Mehdi Mirza,Bing Xu, David Warde-Farley, Sherjil Ozair, AaronCourville and Yoshua Bengio. 2014. Generative Adversarial Nets. In Z. Ghahramani, M.  Welling, C. Cortes, N. D. Lawrence, and K. Q. Weinberger, editors, Advances in Neural Information Processing Systems 27, pages 2672–2680. Curran Associates, Inc.
- Tim Salimans, Ian Goodfellow, Wojciech Zaremba, Vicki Cheung, Alec Radford, Xi Chen, and Xi Chen. 2016. Improved techniques for training gans. In D. D. Lee, M. Sugiyama, U. V. Luxburg, I. Guyon, and R. Garnett, editors, Advances in Neural Information Processing Systems 29, pages 2234–2242. Curran Associates, Inc.
