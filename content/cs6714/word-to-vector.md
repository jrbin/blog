---
title: "Project 2 Word2Vec"
date: 2017-10-17T23:00:11+11:00
---

In this notebook, we demonstrate a basic implementation of Word Embbeddings by training the **skip-gram** model using a small test corpus: **Text8** (based on Tensorflow's word2vec tutorial). It will provide you with hands on experience prior to COMP6714 Project2.

The key part of word embeddings is the training data. In this implementation, mini-batches are generated on demand, and are used by the model to update the word vectors.

You are encouraged to play with this implementation, change model parameters and the training data, and analyze effect of these parameters.

Note that you do not need to dig too deep into the tensorflow and the training part, though if you are into deep learning, it is good to do so.

We evaluate the quality of the embeddings by computing top-10 most nearest words for a sample of words. You will observe that semantically coherent words are embedded close to each other in the embedding space.<br>

Some instructions for this notebook are given as under:<br>

1. We are using **`Tensorflow 1.2.1`**, for this implementation, but this code will work for any version greater than **`1.0`**.
2. For Tensorflow installation, please follow the link: **https://www.tensorflow.org/install/**
3. This notebook will automatically download the training dataset: **'text8.zip'** for Word Embeddings.

Detailed description of each part is given in corresponding sub-sections.

In next cell, we load all the requisite libraries, and modules we'll be using in this notebook.

```python
from __future__ import absolute_import
from __future__ import division
from __future__ import print_function

import collections
import math
import os
import random
from tempfile import gettempdir
import zipfile

import numpy as np
from six.moves import urllib
import tensorflow as tf
from six.moves import range
from six.moves.urllib.request import urlretrieve
from sklearn.manifold import TSNE
```

## Step 1: Downloading the the dataset, and reading it as a list of words.

```python
url = 'http://mattmahoney.net/dc/'

def download(filename, expected_bytes):
    """Download a file if not present in the working directory, and ensure that the file size is correct."""
    if not os.path.exists(filename):
        filename, _ = urlretrieve(url + filename, filename)
    statinfo = os.stat(filename)
    if statinfo.st_size == expected_bytes:
        print('Found and verified %s' % filename)
    else:
        print(statinfo.st_size)
        raise Exception('Failed to verify ' + filename + '. Can you get to it with a browser?')
    return filename

filename = download('text8.zip', 31344016)
print(filename)

# Function to read the data into a list of words.
def read_data(filename):
    """Extract the first file enclosed in a zip file as a list of words."""
    with zipfile.ZipFile(filename) as f:
        data = tf.compat.as_str(f.read(f.namelist()[0])).split()
    return data

data = read_data(filename)
print('Data size', len(data))
```

## Step 2: Build the dictionary and replace rare words with UNK token.

Next celll reads in the data into four global variables:

1. **data** - This variable corresponds to the original text*, i.e.,* the vocabulary of the input corpus. To facilitate processing, we replace the words by corresponding integer ids (*i.e.,* from 0 to vocabulary_size-1).
2. **count** - It is a map *(word_id x count)* used to store the occurences of words in the corpus in order to capture most frequently occuring words.
3. **dictionary** - It is a map *(word x id)* used to store `words` and their integer `ids`.
4. **reverse_dictionary** - It is a map *(id x word)* used to store the inverse mapping of the **dictionary**, to speed the `word/id` look-up process.

```python
vocabulary_size = 50000 # This variable is used to define the maximum vocabulary size.

def build_dataset(words, n_words):
    """Process raw inputs into a dataset.
       words: a list of words, i.e., the input data
       n_words: Vocab_size to limit the size of the vocabulary. Other words will be mapped to 'UNK'
    """
    count = [['UNK', -1]]
    count.extend(collections.Counter(words).most_common(n_words - 1))
    dictionary = dict()
    for word, _ in count:
        dictionary[word] = len(dictionary)
    data = list()
    unk_count = 0
    for word in words:
        index = dictionary.get(word, 0)
        if index == 0:  # i.e., one of the 'UNK' words
            unk_count += 1
        data.append(index)
    count[0][1] = unk_count
    reversed_dictionary = dict(zip(dictionary.values(), dictionary.keys()))
    return data, count, dictionary, reversed_dictionary

data, count, dictionary, reverse_dictionary = build_dataset(data, vocabulary_size)

# del vocabulary  # No longer used, helps in saving space...
print('Most common words (+UNK)', count[:5])
print('Sample data', data[:10], [reverse_dictionary[i] for i in data[:10]])
```

In the next sections, we implement **skip-gram model** for word embeddings. Details about skip-gram model can be found in following paper: `https://arxiv.org/abs/1411.2738`. <br>

## Step 3: Generating training batch for the skip-gram model.

This function is used to generate training batch for the **skip-gram model**. We are using minibatch stochastic-gradient descent to optimize the objective function. For this, we generate a training batch based on following input parameters: **batch_size**, **num_samples**, and **skip_window**.

1. **batch_size** is used to control the size of the `minibatch`* of training data. i.e.,* it is the number of pairs `(CENTER_WORD, CONTEXT_WORD)`. Note that all `CENTER_WORD`s are stored and returned in the list `batch`, and all `CONTEXT_WORD`s are stored and returned in the list `label`.

2. In the `skip-gram` model, we generate training data by capturing the context words from the surrondings of a target word. Here the parameter: **skip_window** defines a sliding window size of size (2\*`skip_window`+1), i.e., up to `skip_window` words to the left and right of the center word.

3. Within a sliding window, we need to sample **num_samples** of context words (i.e., excluding the center word). `num_samples` should be no larger than 2\*`skip_window`.

**Example:**<br>
As an example, with **batch_size**= 8, **skip_window** = 1, and **num_samples** = 2, it generates following data batch. We consider these words as `(input,output)` pairs for the `skip-gram` model.

**Text:** `anarchism originated as a term of abuse first used against early` <br>

**`(CENTER_WORD, CONTEXT_WORD)` Pairs:**<br>
 `(originated,anarchism)`, `(originated,as)`, `(as,originated)`, `(as,a)`, `(a,term)`
 `(a,as)`, `(term,of)`, `(term,a)`

**Explanations:**

1. We choose to always generate `num_skip` pairs for each center word. Therefore,
    * We only need to consider (8/2) center words.
    * We start from the second word (so that it has a full-sized sliding window).
2. With **skip_window** = 1, **num_samples** = 2, we  generate two samples from two context words (in random order).
3. The implementation here is a slightly modified version from tensorflow's word2vec tutorial code; it is still a very ugly implementation.

```python
data_index = 0
# the variable is abused in this implementation.
# Outside the sample generation loop, it is the position of the sliding window: from data_index to data_index + span
# Inside the sample generation loop, it is the next word to be added to a size-limited buffer.

def generate_batch(batch_size, num_samples, skip_window):
    global data_index

    assert batch_size % num_samples == 0
    assert num_samples <= 2 * skip_window

    batch = np.ndarray(shape=(batch_size), dtype=np.int32)
    labels = np.ndarray(shape=(batch_size, 1), dtype=np.int32)
    span = 2 * skip_window + 1  # span is the width of the sliding window
    buffer = collections.deque(maxlen=span)
    if data_index + span > len(data):
        data_index = 0
    buffer.extend(data[data_index:data_index + span]) # initial buffer content = first sliding window

    print('data_index = {}, buffer = {}'.format(data_index, [reverse_dictionary[w] for w in buffer]))

    data_index += span
    for i in range(batch_size // num_samples):
        context_words = [w for w in range(span) if w != skip_window]
        random.shuffle(context_words)
        words_to_use = collections.deque(context_words) # now we obtain a random list of context words
        for j in range(num_samples): # generate the training pairs
            batch[i * num_samples + j] = buffer[skip_window]
            context_word = words_to_use.pop()
            labels[i * num_samples + j, 0] = buffer[context_word] # buffer[context_word] is a random context word

        # slide the window to the next position
        if data_index == len(data):
            buffer = data[:span]
            data_index = span
        else:
            buffer.append(data[data_index]) # note that due to the size limit, the left most word is automatically removed from the buffer.
            data_index += 1

        print('data_index = {}, buffer = {}'.format(data_index, [reverse_dictionary[w] for w in buffer]))

    # end-of-for
    data_index = (data_index + len(data) - span) % len(data) # move data_index back by `span`
    return batch, labels

#
print('data[0:10] = {}'.format([reverse_dictionary[i] for i in data[:10]]))

print('\n.. First batch')
batch, labels = generate_batch(batch_size=8, num_samples=2, skip_window=1)
for i in range(8):
    print(reverse_dictionary[batch[i]], '->', reverse_dictionary[labels[i, 0]])
print(data_index)

print('\n.. Second batch')
batch, labels = generate_batch(batch_size=8, num_samples=2, skip_window=1)
for i in range(8):
    print(reverse_dictionary[batch[i]], '->', reverse_dictionary[labels[i, 0]])
print(data_index)
```

```python
# Specification of Training data:
batch_size = 128      # Size of mini-batch for skip-gram model.
embedding_size = 128  # Dimension of the embedding vector.
skip_window = 1       # How many words to consider left and right of the target word.
num_samples = 2         # How many times to reuse an input to generate a label.
num_sampled = 64      # Sample size for negative examples.
logs_path = './log/'

# Specification of test Sample:
sample_size = 20       # Random sample of words to evaluate similarity.
sample_window = 100    # Only pick samples in the head of the distribution.
sample_examples = np.random.choice(sample_window, sample_size, replace=False) # Randomly pick a sample of size 16

## Constructing the graph...
graph = tf.Graph()

with graph.as_default():

    with tf.device('/cpu:0'):
        # Placeholders to read input data.
        with tf.name_scope('Inputs'):
            train_inputs = tf.placeholder(tf.int32, shape=[batch_size])
            train_labels = tf.placeholder(tf.int32, shape=[batch_size, 1])

        # Look up embeddings for inputs.
        with tf.name_scope('Embeddings'):
            sample_dataset = tf.constant(sample_examples, dtype=tf.int32)
            embeddings = tf.Variable(tf.random_uniform([vocabulary_size, embedding_size], -1.0, 1.0))
            embed = tf.nn.embedding_lookup(embeddings, train_inputs)

            # Construct the variables for the NCE loss
            nce_weights = tf.Variable(tf.truncated_normal([vocabulary_size, embedding_size],
                                                      stddev=1.0 / math.sqrt(embedding_size)))
            nce_biases = tf.Variable(tf.zeros([vocabulary_size]))

        # Compute the average NCE loss for the batch.
        # tf.nce_loss automatically draws a new sample of the negative labels each
        # time we evaluate the loss.
        with tf.name_scope('Loss'):
            loss = tf.reduce_mean(tf.nn.nce_loss(weights=nce_weights, biases=nce_biases,
                                             labels=train_labels, inputs=embed,
                                             num_sampled=num_sampled, num_classes=vocabulary_size))

        # Construct the Gradient Descent optimizer using a learning rate of 0.01.
        with tf.name_scope('Gradient_Descent'):
            optimizer = tf.train.GradientDescentOptimizer(learning_rate = 1).minimize(loss)

        # Normalize the embeddings to avoid overfitting.
        with tf.name_scope('Normalization'):
            norm = tf.sqrt(tf.reduce_sum(tf.square(embeddings), 1, keep_dims=True))
            normalized_embeddings = embeddings / norm

        sample_embeddings = tf.nn.embedding_lookup(normalized_embeddings, sample_dataset)
        similarity = tf.matmul(sample_embeddings, normalized_embeddings, transpose_b=True)

        # Add variable initializer.
        init = tf.global_variables_initializer()


        # Create a summary to monitor cost tensor
        tf.summary.scalar("cost", loss)
        # Merge all summary variables.
        merged_summary_op = tf.summary.merge_all()
```

## Step 5: Begin training

In order to execute the model, we initialize a session object using `tf.Session()`, and call respective node via `session.run()` or `eval()`. General workflow for the training process is as under:

1. Define number of training steps.
2. Initialize all variables*, i.e,* embeddings, weights and biases using `session.run(init)`.
3. Placeholder, (`train_inputs`, and `train_labels`) are used to feed input data to skip-gram using the method `generate_batch`
4. `optimizer`, and `loss` are executed by calling `session.run()`
5. Print out average loss after every **5000** iterations
6. Evaluate the **similarity** after every **10000** iterations. Look for top-10 nearest neighbours for words in sample set.

```python
num_steps = 130001

with tf.Session(graph=graph) as session:
    # We must initialize all variables before we use them.
    session.run(init)
    summary_writer = tf.summary.FileWriter(logs_path, graph=tf.get_default_graph())

    print('Initializing the model')

    average_loss = 0
    for step in range(num_steps):
        batch_inputs, batch_labels = generate_batch(batch_size, num_samples, skip_window)
        feed_dict = {train_inputs: batch_inputs, train_labels: batch_labels}

        # We perform one update step by evaluating the optimizer op using session.run()
        _, loss_val, summary = session.run([optimizer, loss, merged_summary_op], feed_dict=feed_dict)

        summary_writer.add_summary(summary, step )
        average_loss += loss_val

        if step % 5000 == 0:
            if step > 0:
                average_loss /= 5000

                # The average loss is an estimate of the loss over the last 5000 batches.
                print('Average loss at step ', step, ': ', average_loss)
                average_loss = 0

        # Evaluate similarity after every 10000 iterations.
        if step % 10000 == 0:
            sim = similarity.eval() #
            for i in range(sample_size):
                sample_word = reverse_dictionary[sample_examples[i]]
                top_k = 10  # Look for top-10 neighbours for words in sample set.
                nearest = (-sim[i, :]).argsort()[1:top_k + 1]
                log_str = 'Nearest to %s:' % sample_word
                for k in range(top_k):
                    close_word = reverse_dictionary[nearest[k]]
                    log_str = '%s %s,' % (log_str, close_word)
                print(log_str)
            print()

    final_embeddings = normalized_embeddings.eval()
```

## Visualize the Graph, and Loss function in TensorBoard

Open a terminal in current working directory, and follow these steps:<br>
1. Run following command: **tensorboard --logdir=./log/** <br>
2. Open http://127.0.0.0:6006/ into your web browser.<br>
3. Visualize the ** Model Graph** under **GRAPHS** tab.<br>
4. Visualize plot of **loss** under **SCALARS** tab.
