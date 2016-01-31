# VGG Convolutional Neural Networks Practical

*By Andrea Vedaldi and Andrew Zisserman*

This is an [Oxford Visual Geometry Group](http://www.robots.ox.ac.uk/~vgg) computer vision practical, authored by [Andrea Vedaldi](http://www.robots.ox.ac.uk/~vedaldi/) and Andrew Zisserman (Release 2015a).

<img height=400px src="images/cover.png" alt="cover"/>

*Convolutional neural networks* are an important class of learnable representations applicable, among others, to numerous computer vision problems. Deep CNNs, in particular, are composed of several layers of processing, each involving linear as well as non-linear operators, that are learned jointly, in an end-to-end manner, to solve a particular tasks. These methods are now the dominant approach for feature extraction from audiovisual and textual data.

This practical explores the basics of learning (deep) CNNs. The first part introduces typical CNN building blocks, such as ReLU units and linear filters, with a particular emphasis on understanding back-propagation. The second part looks at learning two basic CNNs. The first one is a simple non-linear filter capturing particular image structures, while the second one is a network that recognises typewritten characters (using a variety of different fonts). These examples illustrate the use of stochastic gradient descent with momentum, the definition of an objective function, the construction of mini-batches of data, and data jittering. The last part shows how powerful CNN models can be downloaded off-the-shelf and used directly in applications, bypassing the expensive training process.

[TOC]

$$
   \newcommand{\bx}{\mathbf{x}}
   \newcommand{\by}{\mathbf{y}}
   \newcommand{\bz}{\mathbf{z}}
   \newcommand{\bw}{\mathbf{w}}
   \newcommand{\bp}{\mathbf{p}}
   \newcommand{\cP}{\mathcal{P}}
   \newcommand{\cN}{\mathcal{N}}
   \newcommand{\vc}{\operatorname{vec}}
   \newcommand{\vv}{\operatorname{vec}}
$$

## Getting started

Read and understand the [requirements and installation instructions](../overview/index.html#installation). The download links for this practical are:

* Code and data: [practical-cnn-reg-2016a.tar.gz](http://www.robots.ox.ac.uk/~vgg/share/practical-cnn-reg-2016a.tar.gz)
* Code only: [practical-cnn-reg-2016a-code-only.tar.gz](http://www.robots.ox.ac.uk/~vgg/share/practical-cnn-reg-2016a-code-only.tar.gz)
* Data only: [practical-cnn-reg-2016a-data-only.tar.gz](http://www.robots.ox.ac.uk/~vgg/share/practical-cnn-reg-2016a-data-only.tar.gz)
* [Git repository](https://github.com/vedaldi/practical-cnn) (for lab setters and developers)

After the installation is complete, open and edit the script `exercise1.m` in the MATLAB editor. The script contains commented code and a description for all steps of this exercise, for [Part I](#part1) of this document. You can cut and paste this code into the MATLAB window to run it, and will need to modify it as you go through the session. Other files `exercise2.m`, `exercise3.m`, and `exercise4.m` are given for [Part II](#part2), [III](#part3), and [IV](part4).

Each part contains several **Questions** (that require pen and paper) and **Tasks** (that require experimentation or coding) to be answered/completed before proceeding further in the practical.

## Part 1: CNN bludling blocks {#part1}

In this part we will explore two fundamental bulding blocks of CNNs, linear convolution and non-linear activation functions. Open `exercise1.m` and run the `setup()` command as explained above.

### Part 1.1: convolution {#part1.1}

A *convolutional neural network* (CNN) is a sequence of linear and non-linear  convolutional operators. The most important example of a convolutional operator is *linear convolution*. In this part, we will explore linear convolution and see how to use it in MatConvNet. 

#### Part 1.1.1: convolution by a single filter {#part1.1.1}

Start by identifying and then running the following code fragment in `exercise1.m`:

```.language-matlab
% Load an image and convert it to gray scale and single precision
x = im2single(rgb2gray(imread('data/ray.jpg'))) ;

% Define a filter
w = single([
  0 -1 -0
  -1 4 -1
  0 -1 0]) ;

% Apply the filter to the image
y = vl_nnconv(x, w, []) ;
```

The code loads the image `data/ray.jpg` and applies to it a linear filter using the linear convolution operator. The latter is implemented by the MatConvNet function `vl_nnconv()`. Note that all variables `x`, `w`, and `y` are in single precision; while MatConvNet supports double precision arithmetic too, single precision is usually preferred in applications where memory is a bottleneck. The result can be visualized as follows:

```.language-matlab
% Visualize the results
figure(1) ; clf ; colormap gray ;
set(gcf,'name','P1.1: convolution') ;

subplot(1,3,1) ;
imagesc(x) ;
axis off image ;
title('input image x') ;

subplot(1,3,2) ;
imagesc(w) ;
axis off image ;
title('filter w') ;

subplot(1,3,3) ;
subplot(1,3,3) ;
imagesc(y) ;
axis off image ;
title('output image y') ;
```

> **Task:** Run the code above and examine the result, which should look like the following image:

<img width=80% src="images/conv.png" alt="cover"/>

The input $\bx$ is an $M \times N$ matrix, which can be interpreted as a gray-scale image. The filter $\bw$ is the $3 \times 3$ matrix

$$
\bw = 
\begin{bmatrix}
0 & -1 & 0 \\
-1 & 4 & -1 \\
0 & -1 & 0 \\
\end{bmatrix}
$$

The output of the convolution is a new matrix $\by$ given by[^convolution]
$$
y_{ij} = \sum_{uv} w_{uv}\ x_{i+u,\ j+v}
$$


> **Questions:**
> 
> 1. If $H \times W$ is the size of the input image, $H' \times W'$ the size of the filter, what is the size $H'' \times W''$ of the output image?
> 2. The filter $\bw$ given above is a discretized Laplacian operator. Which type of visual structures (corners, bars, ...) do you think may excite this filter the most?

#### Part 1.1.2: convolution by a filter bank {#part1.1.2}

In neural networks, one usually operates with *filter banks* instead of individual filters. Each filter can be though of as computing a different *feature channel*, characterizing a particular statistical property of the input image.

To see how to define and use a filter bank, create a bank of three filters as follows:

```.language-matlab
% Concatenate three filters in a bank
w1 = single([
  0 -1 -0
  -1 4 -1
  0 -1 0]) ;

w2 = single([
  -1 0 +1
  -1 0 +1
  -1 0 +1]) ;

w3 = single([
  -1 -1 -1
  0 0 0 
  +1 +1 +1]) ;
  
wbank = cat(4, w1, w2, w3) ;
```

The first filter $\bw_1$ is the Laplacian operator seen above; two additional filters $\bw_2$ and $\bw_3$ are horizontal and vertical image derivatives, respectively. Note that the same command `vl_nnconv(x, wbank, [])` works with a filter bank as well. However, the output `y` is not just a matrix, but a 3D  array (often called a *tensor* in the CNN jargon). This tensor has dimensions $H \times W \times K$, where $K$ is the number of *feature channels*.

> **Question:** What is the number of feature channels $C$ in this example? Why?

> **Task:** Run the code above and visualize the individual feature channels in the tensor `y` by using the provided function `showFeatureChannels()`. Do the channel responses make sense given the filter used to generate them?

In a CNN, not only the output tensor, but also the input tensor `x` and the filters `wbank` can have multiple feature channels. In this case, the convolution formula becomes:
$$
y_{ijk} = \sum_{uvp} w_{uvpk}\ x_{i+u,\ j+v,\ p}
$$

> **Questions:** 
> 
> * If the input tensor $\bx$ has $C$ feature channels, what should be the third dimension of $\bw$?
> * In the code above, the command `wbank = cat(4, w1, w2, w3)` concatenates the tensors `w1`, `w2`, and `w3` along the *fourth dimension*. Why is that given that filters should have three dimensions?

#### Part 1.1.3: convolving a batch of data {#part1.1.3}

Finally, in training CNNs it is often important to be able to work efficiently with *batches* of data. MatConvNet allows packing more than one instance of the tensor $\bx$ in a single MATLAB array `x` by stacking instances along the *fourth dimension*:

```.language-matlab
x1 = im2single(rgb2gray(imread('data/ray.jpg'))) ;
x2 = im2single(rgb2gray(imread('data/crab.jpg'))) ;
x = cat(4, x1, x2) ;

y = vl_nnconv(x, wbank, []) ;
```

> **Task:** Run the code above and visualize the result. Convince yourself that each filter is applied to each image.

### Part 1.2: non-linear activation (ReLU) {#part1.2}

CNNs are obtained by composing several operators, individually called *layers*. In addition to convolution and other linear layers, CNNs should contain non-linear layers as well.

> **Question:** What happens if all layers are linear?

The simplest non-linearity is given by scalar activation functions, which are applied independently to each element in a tensor. Perhaps the simples, and one of the most effective, examples is the *Rectified Linear Unit* (ReLU) operator:
$$
   y_{ijk} = \max \{0, x_{ijk}\}
$$
which simply cuts-off any negative value.

In MatConvNet, ReLU is implemented by the `vl_nnrelu` function. To demonstrate its use, we convolve the test image with the negated Laplacian, and then apply ReLU to the result:

```.language-matlab
% Convolve with the negated Laplacian
y = vl_nnconv(x, - w, []) ;

% Apply the ReLU operator
z = vl_nnrelu(y) ;
```

> **Task:** Run this code and visualize images `x`, `y`, and `z`.

> **Questions:** 
> 
> * Which kind of image structures are preferred by this filter? 
> * Why did we negate the Laplacian?

ReLU has a very important effect as it implicitly sets to zero the majority of the filter responses. In a certain sense, ReLU works as a detector, with the implicit convention that a certain pattern is detected when a corresponding filter response is large enough (greater than zero).

In practice, while signals are centered and therefore a threshold of zero is reasonable, in practice there is no particular reason why this should always be appropriate. For this reason, the convolution code allows to specify *a bias term* for each filter response. Let's use this term to make the response of ReLU more selective:

```.language-matlab
bias = single(- 0.2) ;
y = vl_nnconv(x, - w, bias) ;
z = vl_nnrelu(y) ;
```

There is only one `bias` term because there is only one filter in the bank (note that, as for the reset of the data, `bias` is a single precision number). The bias is applied after convolution, effectively subtracting 0.2 from the filter responses. Hence, now a response is not suppressed by the subsequent ReLU operator only if it is at least 0.2.

> **Task:** Run this code and visualize images `x`, `y`, and `z`.

> **Question:** Is the response now more selective?

> **Remark:** There are many other building blocks used in CNNs, the most important of which is perhaps max pooling. However, convolution and ReLU can solve already many problems, as we will see in the remainder of the practical.

## Part 2: backpropagation {#part2}

Training CNNs is normally done using a gradient-based optimization method. The CNN $f$ is the composition of $L$ layers $f_l$ each with parameters $\bw_l$, which in the simplest case of a chain looks like:
$$
 \bx_0
 \longrightarrow 
 \underset{\displaystyle\underset{\displaystyle\bw_1}{\uparrow}}{\boxed{f_1}} 
 \longrightarrow
 \bx_1
 \longrightarrow
 \underset{\displaystyle\underset{\displaystyle\bw_2}{\uparrow}}{\boxed{f_2}}
 \longrightarrow
 \bx_2 
 \longrightarrow
 \dots
 \longrightarrow
 \bx_{L-1}
 \longrightarrow
 \underset{\displaystyle\underset{\displaystyle\bw_2}{\uparrow}}{\boxed{f_L}}
 \longrightarrow
 \bx_L
$$
During learning, the last layer of the network is the *loss function* that should be minimized. Hence, the output $\bx_L = x_L$ of the network is a **scalar** quantity.

The gradient is easily computed using using the **chain rule**. If *all* network variables and parameters are scalar, this is given by[^derivative]:
$$
 \frac{\partial f}{\partial w_l}(x_0;w_1,\dots,w_L)
 =
 \frac{\partial f_L}{\partial x_L}(x_L;w_L) \times
 \cdots
 \times
 \frac{\partial f_{l+1}}{\partial x_l}(x_l;w_{l+1}) \times
 \frac{\partial f_{l}}{\partial w_l}(x_{l-1};w_l) 
$$
With tensors, however, there are some complications. Consider for instance the derivative of a function $\by=f(\bx)$ where both $\by$ and $\bx$ are tensors; this is formed by taking the derivative of each scalar element in the output $\by$ w.r.t. each scalar element in the input $\bx$. If $\bx$ has dimensions $H \times W \times C$ and $\by$ has dimensions $H' \times W' \times C'$, then the derivative contains $HWCH'W'C'$ elements, which is often unmanageable (often of the order of several GBs of memory).

Importantly, all intermediate derivatives in the chain rule are affected by this size explosion, but, since the output is a scalar, the output derivative are not.

> **Question:** The output derivatives have the same size as the parameters in the network. Why?

**Back-propagation** allows computing the output derivatives in a memory-efficient manner. To see how, the first step is to generalize the equation above to tensors using a matrix notation. This is done by converting tensors into vectors by using the $\vv$ (stacking)[^stacking] operator:
$$
 \frac{\partial \vv f}{\partial \vv^\top \bw_l}
 =
 \frac{\partial \vv f_L}{\partial \vv^\top \bx_L} \times
 \cdots
 \times
 \frac{\partial \vv f_{l+1}}{\partial \vv^\top \bx_l} \times
 \frac{\partial \vv f_{l}}{\partial \vv^\top \bw_l} 
$$
The next step is to *project* the derivative with respect to a tensor $\bp_L = 1$ as follows:
$$
 (\vv \bp_L)^\top \times \frac{\partial \vv f}{\partial \vv^\top \bw_l}
 =
 (\vv \bp_L)^\top
 \times
 \frac{\partial \vv f_L}{\partial \vv^\top \bx_L} \times
 \cdots
 \times
 \frac{\partial \vv f_{l+1}}{\partial \vv^\top \bx_l} \times
 \frac{\partial \vv f_{l}}{\partial \vv^\top \bw_l} 
$$
Note that $\bp_L=1$ has the same dimension as $\bx_L$ (the scalar loss) and, being the identity, does not change anything. Things are more interesting when products are evaluated from the left to the right, i.e. *backward from the output to the input* of the CNN. The first such factors is given by:
\begin{equation}
\label{e:factor}
 (\vv \bp_{L-1})^\top = (\vv \bp_L)^\top
 \times
 \frac{\partial \vv f_L}{\partial \vv^\top \bx_L}
\end{equation}
This results in a new projection vector $\bp_{L-1}$, which can then be multiplied to the left to obtain $\bp_{L-2}$ and so on. The last projection $\bp_l$ is the desired derivative. Crucially, each projection $\bp_q$ takes as much memory as the corresponding variable $\bx_q$.

The most attentive reader might have noticed that, while projections remain small, each factor \eqref{e:factor} does contain one of the large derivatives that we cannot compute explicitly. The trick is that CNN toolboxes contain code that can compute the projected derivatives without explicitly computing this large factor. In particular, for any building block function $\by=f(\bx;\bw)$, the toolbox will implement:

* A **forward mode** computing the function $\by=f(\bx;\bw)$.
* A **backward mode** computing the derivatives of the projected function $\langle \bp, f(\bx;\bw) \rangle$ with respect to the input $\bx$ and parameter $\bw$:

$$
\frac{\partial}{\partial \bx} \left\langle \bp, f(\bx;\bw) \right\rangle,
\qquad
\frac{\partial}{\partial \bw} \left\langle \bp, f(\bx;\bw) \right\rangle.
$$

### Part 2.1: Backpropagation interface {#part2.1}

All the building blocks in MatConvNet support forward and backward computation and hence can be used in backpropagation. This is how it looks like for the convolution operator:

```.language-matlab
y = vl_nnconv(x,w,b) ; % forward mode (get output)
p = randn(size(y), 'single') ; % projection tensor (arbitrary)
[dx,dw,db] = vl_nnconv(x,w,b,p) ; % backward mode (get projected derivatives)
```

and this is hat it looks like for ReLU operator:

```.language-matlab
y = vl_nnreli(x) ;
p = randn(size(y), 'single') ;
dx = vl_nnrelu(x,p) ;
```

### Part 2.2: Backward mode for one layer {#part2.2}

Implementing new layers in a network is conceptually simple, but error prone. A simple way of testing a layer is to check whether the derivatives computed using the backward mode  approximately match the derivatives computed numerically using the forward mode. The next example, contained in the file `exercise2.m`, shows how to do this:

```.language-matlab
% Forward mode: evaluate the convolution
y = vl_nnconv(x, w, []) ;

% Pick a random projection tensor
p = randn(size(y), 'single') ;

% Backward mode: projected derivatives
[dx,dw] = vl_nnconv(x, w, [], p) ;

% Check the derivative numerically
delta = 0.01 ;
dx_numerical = zeros(size(dx), 'single') ;
for i = 1:numel(x)
  xp = x ; 
  xp(i) = xp(i) + delta ;
  yp = vl_nnconv(xp,w,[]) ;
  dx_numerical(i) =  p(:)' * (yp(:) - y(:)) / delta ;
end
```

> **Questions:**
> 
> 1.   Recall that the derivative of a function $y=f(x)$ is given by
>     $$
>       \dot f(x) = \lim_{\delta\rightarrow 0} \frac{f(x+\delta) - f(x)}{\delta}
>     $$
>     Can you identify the lines in the code above that use this expression?
> 1.  Note that at line 17 the output value is projected along `p`. Why?


> **Tasks:**
> 
> 1.   Run the code, visualizing the results. Convince yourself that the numerical and analytical derivatives are nearly identical.
> 2.   Modify the code to compute the derivative of the *first element* of the output tensor $\by$ with respect to *all the elements* of the input tensor $\bx$. **Hint:** it suffices to change the value of $\bp$.
> 2.   Modify the code to compute the derivative w.r.t. the convolution parameters $\bw$ instead of the convolution input $\bx$.

### Part 2.3: Backward mode for two or more layers {#part2.3}

Next, we use the backward mode of convolution and ReLU to implement backpropagation in a network that consists of two layers:

```.language-matlab
% Forward mode: evaluate conv followed by ReLU
y = vl_nnconv(x, w, []) ;
z = vl_nnrelu(y) ;

% Pick a random projection tensor
p = randn(size(z), 'single') ;

% Backward mode: projected derivatives
dy = vl_nnrelu(z, p) ;
[dx,dw] = vl_nnconv(x, w, [], dy) ;
```

> **Question (important)** In the code above, in backward mode the projection `p` is fed to the `vl_nnrelu` operator. However, the `vl_nnconv` operator now receives `dy` as projection. Why?

> **Tasks:**
>
> 1.  Run the code and visualize the analytical and numerical derivatives. Do they differ?
> 2.  (Optional) Modify the code above to a chain of three layers: conv + ReLU + conv.

### Part 2.4: Implementing a custom layer

Creating new layers is a common task when testing novel CNN architectures. In this part you will implement a layer computing the Euclidean distance between a tensor `x` and a reference tensor `r`. This layer will be used later to learn a CNN from data.

The first step is to write the forward mode. This is contained in the `customLayerForward.m` function. Open the file and check its content:

```.language-matlab
function y = customLayerForward(x, r)
y = sum(sum(sum((x - x0).^2, 1), 2), 3) ;
```

The function computes the difference `x - r`, squares the individual elements (`.^2`), and then sums the result along the first, second, and third dimensions. The result is a tensor `y` which has dimension $1 \times 1 \times 1 \times N$ (a scalar for each of the $N$ data instances in the batch) and value equal to the squared Euclidean distance between `x` and `x0`.

Next, we need to implement the backward mode as well:

```.language-matlab
function dx = customLayerBackward(x,r,p)
dx = 2 * bsxfun(@times, p, x - r) ;
```

Note that the backward mode takes the projection tensor `p` as an additional argument. Let us show that this code is correct. Recall that the goal of the backward mode is to compute the derivative of the projected function:

$$
\langle \bp, f(\bx) \rangle
= p_{1,1,1,t} \sum_{lmn} (x_{lmnt} - r_{lmnt})^2.
$$

Here the subscript $t$ index the data instance in the batch; note that, for a given $t$, the projection is simply a scalar, since the output of the Euclidean distance is also a scalar.

Next, we compute the derivative w.r.t. each input element $x_{ijkt}$:

$$
\frac{\partial}{\partial x_{ijk}}
\langle \bp, f(\bx) \rangle
= 2 p_{1,1,1,t} (x_{ijkt} - r_{ijkt}).
$$

In the code, the `bsxfun` MATLAB function is used in order to multiply the value $p_{1,1,1,t}$ with all the elements $x_{ijkt} - r_{ijkt}$ in a single step.

> **Tasks:**
> 
> 1.  Verify that the forward and backward functions are correct by computing the derivatives numerically.
> 2.  Change the code such that the Euclidean distance is averaged instead of being summed across spatial location (**Hint:** simply divide by the product of `size(x,1)` and `size(x,2)`).
> 3.  Make sure that both the forward and backward functions are correctly modified by verifying the result numerically once more.

## Part 3: Learning a CNN for text deblurring {#part3}

By now you should be familiar with two basic CNN layers, convolution and ReLU, as well as with the idea of backpropagation. In this part, we will build on such concepts to learn a CNN model.

CNN are often used for classification; however, they are much more general than that. In order to demonstrate their flexibility, here we will design a CNN that takes an image as input and produces an image as output (instead of a class label).

We will consider in particular the problem of *deblurring images of text*, as in the following example:

![Data example](images/text.png)

### Part 3.1: preparing the data {#part3.1}

The first task is to load the training and validation data and to understand its format. Start by opening in your MATLAB editor `exercise3.m`. The code responsible for loading the data is

```.language-matlab
imdb = load('data/text_imdb.mat') ;
```

The variable `imdb` is a structure containing $n$ images, which will be used for training and validation. The structure has the following fields:

* `imdb.images.data`: a $64 \times 64 \times 1 \times n$ array of grayscale blurred images.
* `imdb.images.label`: a $64 \times 64 \times 1 \times n$ of grayscale sharp images.
* `imdb.images.set`: a $1 \times n$ vector containing a 1 for training images and an 2 for validation images. 75% of the images are used for training and 25% for test.

Run the following code, which displays the first image in the dataset and its label:

```.language-matlab
figure(100) ; clf ;

subplot(1,2,1) ; imagesc(imdb.images.data(:,:,:,1)) ;
axis off image ; title('input (blurred)') ;

subplot(1,2,2) ; imagesc(imdb.images.label(:,:,:,1)) ;
axis off image ; title('desired output (sharp)') ;

colormap gray ;
```

> **Task:** make sure you understand the format of `imdb`. Use MATLAB to find out the number of training and validation images as well as the resolution (size) of each image.

It is often important to center the data to better condition the learning problem. This is usually obtained by subtracting the mean pixel intensity (computed from the training set) from each pixel. Here, however, images are rescaled to have values in the interval $[-1, 0]$.

> **Question:** why was the interval $[-1, 0]$ chosen? **Hint:** what intensity corresponds to 'white'? What does the convolution operator do near the image boundaries?

### Part 3.2: defining a CNN architecture

Here we define a CNN `net` and initialize its weights randomly. A CNN is simply a collection of interlinked layers. While these can be assembled 'manually' as you did in Part 2, it is usually more convenient to use a **wrapper**.

MatConvNet contains two wrappers, SimpleNN and DagNN. SimpleNN is suitable for simple networks that are a chain of layers (as opposed to a more general graph). We will use SimpleNN here.

This wrapper defines the CNN as a structure `net` containing a cell-array `layers` listed in order of execution. Open `initializeSmallCNN.m` and find this code:

```.language-matlab
net.layers = { } ;
```

The first layer of the network is a convolution block:

```.language-matlab
net.layers{end+1} = struct(...
  'name', 'conv1', ...
  'type', 'conv', ...
  'weights', {xavier(3,3,1,32)}, ...
  'pad', 1, ...
  'learningRate', [1 1], ...
  'weightDecay', [1 0]) ;
```

The fields are as follows:

* `name` specifies a name for the layer, useful for debugging but otherwise arbitrary. 

* `type` specifies the layer type, in this case convolution. 

* `weights` is a cell array containing the layer parameters, in this case two tensors for the filters and the biases. The filters are initialized using the `xavier` function to have dimensions $3 \times 3 \times 1 \times 32$ ($3\times 3$ spatial support, 1 input feature channels, and 32 filters). The function also initializes the biases to be zero.

* `pad` specifies the amount of zero padding to apply to the layer input. By using a padding of one pixel and a $3\times 3$ filter support, the output of the convolution will have exactly the same height and width as the input.

* `learningRate` contains two layer-specific multipliers to adjust the learning rate for the filters and the biases.

* `weightDecay` contains two layer-specific multipliers to adjust the weight decay (regularization strength) for the layer filters and biases. Note that weight decay is not applied to the biases.

> **Question:** what would happen `pad` was zero?

The convolution layer is followed by ReLU, which is given simply by:

```.language-matlab
net.layers{end+1} = struct(...
  'name', 'relu1', ...
  'type', 'relu') ;
```

This pattern is repeated (varying the number and dimensions of filters) for a total of three convolutional layers separated by ReLUs.

> **Question:** The last layer, generating the output image, is convolutional and is *not* followed by ReLU. Why?

The command `vl_simplenn_display()` can be used to print information about the network. Here is a subset of this information:

|      layer|    0|    1|    2|    3|    4|         5|     6|
|:---------:|:---:|:---:|:---:|:---:|:---:|:--------:|:----:|
|       type|input| conv| relu| conv| relu|      conv|custom|
|       name|  n/a|conv1|relu1|conv2|relu2|prediction|  loss|
|    support|  n/a|    3|    1|    3|    1|         3|     1|
|   filt dim|  n/a|    1|  n/a|   32|  n/a|        32|   n/a|
|  num filts|  n/a|   32|  n/a|   32|  n/a|         1|   n/a|
|     stride|  n/a|    1|    1|    1|    1|         1|     1|
|        pad|  n/a|    1|    0|    1|    0|         1|     0|
|    rf size|  n/a|    3|    3|    5|    5|         7|     7|

> **Questions:** Look carefully at the generated table and answer the following questions:
>
> 1. How many layers are in this network?
> 2. What is the support (height and width) and depth (number of feature channels) of each intermediate feature map?
> 3. How is the number of feature channels related to the 
>    dimensions of the filters?

The last row reports the *receptive field size* for the layer. This is the size (in pixels) of the local image region that affects a particular element in a feature map. 

> **Question:**  what is the receptive field size of the pixel in the output image (generated by the prediction layer)? Discuss whether a larger receptive field size might be preferable for this problem.

### Part 3.3: learning the network {#part3.3}

In this part we will use SGD to learn the CNN from the available training data. As noted above, the CNN must however terminate in a loss layer. We add one such layer as follwos:

```.language-matlab
% Add a loss (using our custom layer)
net.layers{end+1} = getCustomLayer() ;
```
The function `getCustomLayer()` creates a `layer` structure compatible with SimpleNN. This structure contains handles to the functions defined in Part 2, namely `customLayerForward()` and `customLayerBackward()`. 

> **Remark:** If your implementation of these functions is incorrect, the next few steps will fail!

Next, we setup the learning parameters:

```.language-matlab
trainOpts.expDir = 'data/text-small' ;
trainOpts.batchSize = 16 ;
trainOpts.learningRate = 0.01 ;
trainOpts.numEpochs = 30 ;
trainOpts.gpus = [] ;
trainOpts.errorFunction = 'none' ;
```

The fields are ass follows:

* `expDir` specifies a directory to store intermediate data (snapshot and figures) as well as the final model. Note that the code resumes execution from the last snapshot; therefore change this directory or clear it if you want to start learning from scratch.

* `batchSize` specifies how many images to include in a batch. Here we use 16.

* `learningRate` is the learning rate in SGD.

* `numEpochs` is the number of epochs (passes through the training data) to perform before SGD stops.

* `gpus` contains a list of GPU IDs to use. For now, we do not use any.

* `errorFunction` disable plotting the default error functions that are suitable for classification, but not for our problem.

Finally, we can invoke the learning code:

```
net = cnn_train(net, imdb, @getBatch, trainOpts) ;
```

The `getBatch()` function, passed as a *handle*, is particularly important. The training script `cnn_train` uses `getBatch()` to extract the images and corresponding labels for a certain batch, as follows:

```.language-matlab
function [im, label] = getBatch(imdb, batch)
im = imdb.images.data(:,:,:,batch) ;
label = imdb.images.label(:,:,:,batch) ;
```

The function takes as input the `imdb` structure defined above and a list `batch` of image indexes that should be returned for training. In this case, this amounts to simply extracting and copying some data; however, in general `getBatch` can be used to e.g. read images from disk or apply transformations to them on the fly.

## Part 3.4: evaluating the network

The network is evaluated on the validation set during training. The validation error (which in our case is the average squared differences of the predicted output pixels and the desired ones), is a good indicator of how well the network is doing (in practice, one should ultimately evaluate the network on a held-out test set).

However, in our example it is also informative to evaluate the *qualitative* result. This can be done as follows:

```.language-matlab

train = find(imdb.images.set == 1) ;
val = find(imdb.images.set == 2) ;

figure(101) ; set(101,'name','Resluts on the training set') ;
showDeblurringResult(net, imdb, train(1:30:151)) ;

figure(102) ; set(102,'name','Resluts on the validation set') ;
showDeblurringResult(net, imdb, val(1:30:151)) ;
```

> **Questions:** 
> 
> * Do you think the network is doing a good job?
> * Is there any obvious difference between training and validation performance?

### Part 3.5: Learning a larger model using the GPU

So far, we have trained a single small network to solve this problem. Here, we will experiment with several variants to try to improve the performance as much as possible.

Before we experiment further, however, it is beneficial to switch using a GPU. If you have a GPU and MATLAB Parallel Toolbox installed, you can try running the code on a GPU by changing a single switch. Assuming that the GPU has index 1 (which is always the case if there is a single CUDA-compatible GPU in your machine):

```
trainOpts.expDir = 'data/text-small-gpu'
trainOpts.gpus = [1] ;
```

Note that we change `expDir` in order to start a new experiment from scratch.

> **Task:** Test GPU based training (if possible). How much faster does it run?

Now we are ready to experiment with different CNNs. 

> **Task:** Run a new experiment, this time using the `initializeLargeCNN()` function to construct a larger network.

> **Questions:**
> 
> 1.  How much slower is this network compared to the small model?
> 2.  What about the quantitative performance on the validation set?
> 3.  What about the qualitative performance?

### Part 3.6: Challenge!

You are now in control. Play around with the model definition and try to improve the performance as much as possible. For example:

* Try adding more layers.
* Try adding more filters.
* Try increasing the receptive field size by increasing the filter support (do not forget to adjust the padding).
* Try sequences of rank-1 filters, such as $7 \times 1$ followed by $1 \times 7$ to increase the receptive field size while maintaining efficiency.

And, of course, make sure to beat the other students.

## Links and further work

* The code for this practical is written using the software package [MatConvNet](http://www.vlfeat.org/matconvnet). This is a software library written in MATLAB, C++, and CUDA and is freely available as source code and binary.

* MatConvNet can train complex computer vision models, such as VGG VD and Inception. Several of these models, including a few cool demos, are available for download.

* Many more computer vision practicals are available [here](https://www.robots.ox.ac.uk/~vgg/practicals/overview/index.html).

## Acknowledgements

* Beta testing by: Karel Lenc and Carlos Arteta.

## History

* Used in the XXX, Matla, 2016..

[^convolution]: if you are familiar with convolution as defined in mathematics and signal processing, you might expect to find the index $i-u$ instead of $i+u$ in this expression. The convention $i+u$, which is often used in CNNs, is  often referred to as correlation.

[^derivative]: The derivative is computed with respect to a certain assignment $x_0$ and $(w_1,\dots,w_L)$ to the network input and parameters; furthermore, the intermediate derivatives are computed at points $x_1,\dots,x_L$ obtained by evaluating the network at $x_0$.

[^stacking]: The stacking operator $\vv$ simply unfolds a tensor in a vector by stacking its elements in some pre-defined order.

[^lattice]: A two-dimensional *lattice* is a discrete grid embedded in $R^2$, similar for example to a checkerboard.