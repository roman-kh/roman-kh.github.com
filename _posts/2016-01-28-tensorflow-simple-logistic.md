---
layout: post
title: Simple logistic regression with Tensorflow
description: "Your python programs are realy slow unless you use Numba. Part 2."
tags: [python, tensorflow, logistic]
date: 2016-01-28
image:
  feature: abstract-7.jpg
---

After a long struggle I managed to build from sources Tensorflow for GPU with CUDA capability=3.0.
So now I can dig deeper into what Tensorflow is and how one can solve analytics tasks with it.

Let's begin with a logistic regression, a simple, yet pretty powerful tool suitable for real-life business problems.

{% highlight python %}
import tensorflow as tf

# to begin with, load your data
# you have to implement your own load_data function
train_X, train_Y, test_X, test_Y = load_data()

# data format is as usual:
# train_X and test_X have shape (num_instances, num_features)
# train_Y and test_Y have shape (num_instances, num_classes)
num_features = train_X.shape[1]
num_classes = train_Y.shape[1]

# Create variables
# X is a symbolic variable which will contain input data
# shape [None, num_features] suggests that we don't limit the number of instances in the model
# while the number of features is known in advance
X = tf.placeholder("float", [None, num_features])
# same with labels: number of classes is known, while number of instances is left undefined
Y = tf.placeholder("float",[None, num_classes])

# W - weights array
W = tf.Variable(tf.zeros([num_features,num_classes]))
# B - bias array
B = tf.Variable(tf.zeros([num_classes]))

# Define a model
# a simple linear model y=wx+b wrapped into softmax
pY = tf.nn.softmax(tf.matmul(X, W) + B)
# pY will contain predictions the model makes, while Y contains real data

# Define a cost function
cost_fn = tf.reduce_mean(tf.nn.softmax_cross_entropy_with_logits(pY, Y))
# You could also put it in a more explicit way
# cost_fn = -tf.reduce_sum(Y * tf.log(pY))

# Define an optimizer
# I prefer Adam
opt = tf.train.AdamOptimizer(0.01).minimize(cost_fn)
# but there is also a plain old SGD if you'd like
#opt = tf.train.GradientDescentOptimizer(0.01).minimize(cost_fn)

# Create and initialize a session
sess = tf.Session()
init = tf.initialize_all_variables()
sess.run(init)

num_epochs = 40
for i in range(num_epochs):
  # run an optimization step with all train data
  sess.run(opt, feed_dict={X:train_X, Y:train_Y})
  # thus, a symbolic variable X gets data from train_X, while Y gets data from train_Y

# Now assess the model
# create a variable which reflects how good your predictions are
# here we just compare if the predicted label and the real label are the same
accuracy = tf.reduce_mean(tf.cast(tf.equal(tf.argmin(pY,1), tf.argmax(Y,1)), "float"))
# and finally, run calculations with all test data
accuracy_value = sess.run(accuracy, feed_dict={X:test_X, Y:test_Y})
{% endhighlight %}
