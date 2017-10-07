% Tensorflow Manifesto
% Daniel Waterworth
% October 7, 2017

![](../images/fight.jpg)

_image from used under creative commons, [see original](https://www.flickr.com/photos/39400215@N04/3713813916/)_

## The way things are

Tensorflow is the most widely used machine learning framework. It's rise has
been nothing short of meteoric. It's used by everyone from [Japanese cucumber
farmers](https://cloud.google.com/blog/big-data/2016/08/how-a-japanese-cucumber-farmer-is-using-deep-learning-and-tensorflow)
(supposedly) to the team behind [alphago](https://deepmind.com/research/alphago/).

You might reasonably think then, with all this code written against this
particular framework, that code-sharing and reuse within the tensorflow
ecosystem would be rife. Unfortunately, and as you will undoubted know if you
have but dabbled in these waters, this is not the case. The ecosystem is
fragmented beyond reason. There are more high-level libraries than I care to
list and they don't play well together. Many were created by google themselves.
Looking for an implementation of a particular model? It's not enough that the
authors used tensorflow; it needs to be in your tensorflow strain.

This state of affairs serves nobody and it's time we did something to fix it.
What I am proposing is not another meta-framework, as if the problem would be
solved if everyone just did things my way™. Instead, I propose a set of
principles that will facilitate sharing, reuse and co-mingling. Perhaps, by
adherence to these ideals, all of tensorflow-dom can finally be united.

My central thesis is that the reason for the current fragmentation; the reason
existing meta-frameworks don't play nicely together is, in a word, variables.
Each library manages them differently and in incompatible ways. If we could
agree on how we manage our variables, perhaps we could mend our fragmented
ecosystem.

## Principles

So, without further ado, my principles for a united tensorflow are:

 1. All variables must by owned by an object,
    * This object:
        - must be capable of initializing its variables within a session,
        - must provide, upon request, a list of all of its variables; this
          facilitates, amongst other things, saving/loading,

## Demostration

Allow me to demonstrate this style with a short, but complete, example. Here,
we'll be fine-tuning a pretrained model on a different task. We'll just retrain
the top layer.

```python
import tensorflow as tf
import somelib

def my_network(params, input):
    output = somelib.conv2d(params['conv1'], input, 50, (3, 3))
    return output

params = somelib.Parameters('my_network')

input = tf.placeholder('input', [batch_size, 64, 64, 3])
expected_output = tf.placeholder('expected_output', [batch_size, 1000])

# The normal optimizers in tensorflow don't keep track of their variables, but
# this one does.
optimizer = somelib.Optimizer(loss, params.variables)

with tf.Session() as sess:
    optimizer.initialize()
    params.load_from('pretrained_network.model')

    for i in range(1000):
        inputs, outputs = load_batch()
        loss_value, _ = \
            sess.run(
                [loss, optimizer.train_op],
                feed_dict={input: inputs, expected_output: outputs}
            )
        print(loss_value)

    params.save_to('fine_tuned_network.model')
```
