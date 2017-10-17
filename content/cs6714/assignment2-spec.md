---
title: "Project 2 Specification"
date: 2017-10-17T22:37:11+11:00
---

* ver 1.0.0
* Last updated: 17 Oct 2017

### Submission Deadline: **13 Nov, 2017 (23:59:59 PM )**

In this Project, you will be implementing your own Word Embeddings for adjectives. More specifically, we want the obtained embeddings to preserve as much synonym relationship as possible. This will be measured against a set of ground truth assembled manually from dictionaries. Key instructions concerning the project are listed below:

* The model training required in the project is **time-consuming** (even if you have a high-end GPU card), so it is highly advised that you start working on this as early as possible.
    * It may take up to several hours to train your embeddings once using `CPU`.
* You are supposed to implement your methodology using Tensorflow (version > **1.0**).
* You are **not** allowed to use supervised approaches to improve the model performance and you are **not** allowed to hard code the ground truth into your model.
* You are required to implement your code using Python3.
* You are required to use `spaCy(version greater than 1.8.0)` to process input data (not `nltk`). You can assume `spaCy` always gives the correct parsing results.
* You will only be working with the provided dataset named: `BBC_Data.zip`, you are not supposed to augment this data using external/additional data sources.


# 1: INTRODUCTION

The project constitutes two major parts listed below:

## 1.1: Data Processing and Training Data Generation

We provide only the raw form of data to be used in this project*, i.e.,* `BBC_Data.zip`. For embeddings, you are required to process this dataset using `spaCy`. You can write out your processed data file in the Present Working Directory (PWD), which can later be used to train the embeddings.

While the vanilla word2vec skip-gram model generates its (mini-batches of) training data between a center word and a random sample of its context, you are encouraged to think of new methods such that the resulting embedding for adjectives are more likely to preserve the synonym relationship in its top-k results.

## 1.2: Model Training:

Model training part is supposed to read out your processed data, and come up with best possible settings to train the embeddings. For training, we have categorized the model parameters into two categories namely:*(i)*`fixed` and *(ii)*`tunnable`. You are supposed to come up with the best settings for `tunable` parameters, keeping the `fixed` parameters as such.

### 1.2.1 Tunable Parameters:

You are required to fine-tune the following parameters (1-6):

1. **batch_size** <br>
`batch_size` defines the size of minibatch used at a time to update the embeddings. You are allowed to come up with best value for batch size, its value should be less than 128.

2. **skip_window** <br>
`skip_window` defines the size of the context window surronding the target word, as illustrated in the notebook `Word2Vec_Demo.ipynb`.

3. **num_samples** <br>
`num_samples` defines the number of words to sample from the `skip_window`, as illustrated in the notebook `Word2Vec_Demo.ipynb`.

4. **Vocabulary_size** <br>
Maximum vocabulary size of your embeddings, you should adjust this parameter considering statistics of your processed data (i.e., after processing the raw data `BBC_Data.zip`).

5. **learning_rate for optimizer** <br>
`learning_rate` defines the learning rate of the optimizer. You should start with a low value (~ 0.001), and gradually increase it.

6. **Number of Negative Samples** <br>
Number of negative samples for the Sampled_Softmax loss, as illustrated in the notebook `Word2Vec_Demo.ipynb`.


### 1.2.2 Fixed Parameters:

To simplify experimentation, we will fix values for following parameters:

1. **Embedding_dimensions** <br>
You will be using 200-dimensions for the embeddings

2. **Number of iterations** <br>
You will be training your model with 100,001 training iterations each of batch_size less than or equal to 128.

3. **Loss function** <br>
You will be using `sampled_softmax_loss`, you are encouraged to see how is it different from Noise-Contrastive Estimation implemented in `Word2Vec_Demo.ipynb`

4. **Optimization Method** <br>
You will be using `AdamOptimizer` with best possible value for the learning rate. <br>

**Note**

* You are required to come up with best settings for `tunable` parameters, while keeping the values for `fixed` parameters as such.

## 1.3 Recommendations

Section 1.1 should be the main focus of the project and we encourage you to experiment with different choices of training data for the word2vec skip-gram model. You shall learn both expected and unexpected lessons throughout this kind of interactions. In this direction, you may find (1) version and provenance control, and (2) batch execution systems useful. For example, you may want to revert your method to one that tried 3 days ago with a particular set of parameters and options; or you may want to do a grid search of possible parameter values automatically over the night, :)

# 2: Submission:

You are required to complete your implementation in the file: `submission.py`.<br>

You should : <br>
(a) Complete the method `process_data(input_data)`. This method:<br>

* Reads the raw data `BBC_Data.zip`
* Processes the data and writes out the processed file in Present Working Directory(PWD).<br>

(b) Complete the method `adjective_embeddings(data_file, embeddings_file_name, num_steps, embedding_dim)` in the file: *submission.py*, using optimal settings for tunable parameters(1-6). This method should: <br>

* Train embeddings using processed data from Part (2-a)
* Write out trained embeddings in the file `"adjective_embeddings.txt"`.
* You should store each float in the `"adjective_embeddings.txt"` upto 6 decimal places.

(c) Complete the method `Compute_topk(model_file, input_adjective, top_k)`. This method should:

* Read the `model_file` using python library gensim
* Read the `input_adjective`.
* Return **top_k** most_similar words (synonyms) for the `input_adjective`.

(d) **You will be submitting**

* Your complete implementation in the file `submission.py`.
* Trained embeddings in a file named: `"adjective_embeddings.txt"` .
* A report outlining your methodology, and reasoning and experimental results to support your methodology.

Total file size is capped at **100MB**.

**Note:** _Your trained embeddings model named: `"adjective_embeddings.txt"` should be loadable using the python library `gensim` (explained in "5 Evaluation"). **In case, if your model fails to load, you will recieve very low score.**_

```python
# In this section, we load all the requisite libaries.
import os
import math
import random
import zipfile
import numpy as np
import tensorflow as tf
```

# 3: Reading and Processing Input data

In this part, you will be reading the data file named: `"BBC_Data.zip"` from the working directory. This file contains sub-directories collected from BBC News named: (a) business, (b) entertainment, (c) politics, (d) sports, and (e)tech. <br>

You are required to read all the files in `BBC_Data.zip`, process the data using spaCy, and write out your processed data in a seperate file.

**Note:** You are allowed to write only one data file.

```python
import submission as submission
input_dir = './BBC_Data.zip'
data_file = submission.process_data(input_dir)
```

# 4: Model training

In this part you are supposed to read the processed data, train your model, and write out your embeddings in the file: `"adjective_embeddings.txt"`.

```python
import submission as submission

## Output file name to store the final trained embeddings.
embedding_file_name = 'adjective_embeddings.txt'

## Fixed parameters
num_steps = 100001
embedding_dim = 200


## Train Embeddings, and write embeddings in "adjective_embeddings.txt"
submission.adjective_embeddings(data_file, embedding_file_name, num_steps, embedding_dim)
```

# 5: Evaluation

We will load your trained model `"adjective_embeddings.txt"`, and test it using the following set of adjectives: <br>

```
Test_Adjectives =[
'able', 'average', 'bad', 'best', 'big', 'certain', 'common', 'current', 'different',
'difficult', 'early', 'extra', 'fair', 'few', 'final', 'former', 'great', 'hard',
'high', 'huge', 'important', 'key', 'large', 'last', 'less', 'likely',
'little', 'major', 'more', 'most', 'much', 'new', 'next', 'old', 'prime', 'real', 'recent',
'same', 'serious', 'short', 'small', 'top', 'tough', 'wide']
```

We evaluate your model by:

1. Selecting one adjective at a time from the list `Test_Adjectives`.
2. For the selected adjective, computing a list of **top_k** most nearest words using the method `Compute_topk(model_file, input_adjective, top_k)`
3. Comparing your output for the adjective with ground truth list of synonyms for that adjective, and evaluating **Hits@k(k = 100)**.
4. Average out the result **(Hits@k(k= 100))** for all the adjectives to calculate Average Precision.


### Evaluation Example (using a smaller `k` value):

a) <br>
Ground_truth_new = ['novel', 'recent', 'first', 'current', 'latest'] <br>
Output_new = ['old', 'novel', 'first', 'extra', 'out'] <br>
Hits@k(k=5) = 2

b) <br>
Ground_truth_good = ['better', 'best', 'worthwhile', 'prosperous', 'excellent'] <br>
Output_good = ['hate', 'better', 'best', 'worthwhile', 'prosperous'] <br>
Hits@k(k=5) = 4

Average_Hits@k(k=5) = (2+4)/2 = 3.0

## 5.1 Loading Pre-trained model using Gensim

Before final submission, you should check that you can load your trained model `"adjective_embeddings.txt"` using `gensim`, and test that it supports the operation `model.most_similar()` as illustrated below.

As an example, in this section, we load a pre-trained file `"adjective_embeddings.txt"` using `gensim`.

```python
import gensim

model_file = 'adjective_embeddings.txt'

## How we will load your trained embeddings for evaluation.
model = gensim.models.KeyedVectors.load_word2vec_format(model_file, binary=False)
```

## 5.2: Computing Top-k results

To evaluate your results, we will run the method named: `Compute_topk(model_file, input_adjective, top_k)` that reads in:<br>

* pre-trained model file `"adjective_embeddings.txt"` using gensim
* an input adjective `input_adjective`
* top_k

and returns a list of `top_k` similar words for the `input_adjective`.

**Note :** *The method `Compute_topk(model_file, input_adjective, top_k)` has been added to allow post-processing on embeddings results.*

```python
import gensim
import submission as submission

model_file = 'adjective_embeddings.txt'
input_adjective = 'bad'
top_k = 5

output = []
output = submission.Compute_topk(model_file, input_adjective, top_k)
print(output)
```

## 5.3 Sample Adjectives and their Synonyms

For experimentation, we provide you with a sample list of adjectives alongside their synonyms.
These synonyms lists are not complete, they just denote a small subset of the possible synonyms of each adjective that you may use to evaluate your embeddings.

````python
Sample_adj = ['new', 'more', 'last', 'best', 'next', 'many', 'good']

Sample_adj_synonyms = {}

Sample_adj_synonyms['new'] = ['novel', 'recent', 'first', 'current', 'latest',...]
Sample_adj_synonyms['more'] = ['higher' , 'larger', 'unlimited', 'greater', 'countless',...]
Sample_adj_synonyms['last'] = ['past' , 'final', 'previous', 'earlier', 'late',...]
Sample_adj_synonyms['best'] = ['finest' , 'good', 'greatest', 'spotless', 'perfect',...]
Sample_adj_synonyms['next'] = ['coming' , 'following', 'forthcoming', 'consecutive', 'upcoming',...]
Sample_adj_synonyms['many'] = ['most' , 'countless', 'excess', 'several', 'much',...]
Sample_adj_synonyms['good'] = ['better', 'best', 'worthwhile', 'prosperous', 'fantastic',...]
````

## 5.4 Late penalty

**-10% per day for the first three days, and -20% per day afterwards.**
