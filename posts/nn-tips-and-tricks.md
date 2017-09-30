% NNs: Tips and Tricks
% Daniel Waterworth
% September 30, 2017

## Background

I been interested in machine learning since before it was cool, but it's only
relatively recently that I've really begun training my own neural networks. So,
whilst it's all fresh in my mind, I want to jot down some notes that may be
helpful for my future self and others. I intend to update this document in the
future as I discover new things.

## 1. Padding

In my experience, padding causes more problems than it solves. When you pad
before convolving, you are forcing your convolutions to handle two different
cases with the same weights; the case where the convolution window is somewhere
in the centre and the case where it is hanging over an edge. You can mitigate
this somewhat by adding a channel of 1s before padding, but with a little
thought, its possible to redesign your neural network so that it doesn't pad at
all and the results tend to be much better.

## 2. Transposed Convolutions

Again, avoid. Transposed convolutions tend to produce blocking artefacts. This
effect is well documented [^1]. When you need to scale up, use nearest neighbor
or bilinear upsampling.

## 3. Strided Convolutions

Think of a strided convolution as the combination of a dense convolution and an
operation that throws away alternate rows and columns. The information in all
those rows and columns may have been useful. My advice is: allow the neural
network to decide what information to keep. When you need to scale down
spatially, do a space-to-depth conversion and potentially follow that with a
1x1 convolution to reduce the number of channels.

[^1]: [Deconvolution and Checkerboard Artifacts](https://distill.pub/2016/deconv-checkerboard/)
