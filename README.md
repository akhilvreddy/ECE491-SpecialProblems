# ECE491, Special Problems

## Introduction
The main goal of this research is to recover signals in the presence of Speckle Noise. The way this can be done is using Machine Learning, Autoencoders, and algorithms based off that. My approach, code, algorithms, and process will be outlined below.

### Motivation 
There are a wide range of computational imaging systems that illuminate the object of interest
by coherent light. Examples of such imaging systems are synthetic aperture radar (SAR) and
digital holography. These systems all suffer from a very particular type of noise referred to as
speckle noise. Speckle noise is very different and than typical additive noise, since it is
multiplicative. Most solutions to tackle speckle noise seem to be heuristic. The main motivation
for this work is the recent paper by Prof. Jalali and her collaborators that looks into such
systems from a theoretical perspective and shows that compressed sensing in the presence of
speckle noise is possible. However, the results of that paper are mainly theoretical. In this
project, we are interested in taking advantage of powerful tools from machine learning, such as
auto encoders, to implement algorithms that recover a signal in the presence of speckle noise.

### Concepts / Tools being used
The problem of signal recovery from noisy samples that are corrupted by speckle noise is
inherently very complex mathematically. To be successful in this project, I need to understand
those concepts and then, based on the results of that paper and other prior work on application
of compressed sensing, implement a proper recovery algorithm.

Most of my coding will be done in Python3 and the main libraries that I will be using are Numpy,
Numba, and PyTorch. Numba is going to be used for a wide variety of operations such as
speeding up calculations and will translate the code to be easily-compilable. Numpy is the basic
mathematical calculation library that I will be using to work with doing operations on vectors.
PyTorch is the biggest library that I will use since it is the main machine learning tool that I have.
I’ll be using that to train and use neural networks that I have to make. Also, PyTorch is an
important skill to learn as an ECE undergraduate because of its wide use and doing this project
would help me.

## Thesis 
The use of advanced signal processing techniques, such as denoising combined with machine learning algorithms can significantly improve the accuracy of signal recovery from noisy image samples of lungs with pneumonia, leading to more accurate diagnosis and treatment of the disease.

## The problem

Speckle noise is a type of noise that is commonly found in images acquired from optical imaging systems, such as laser imaging, ultrasound imaging and synthetic aperture radar (SAR) imaging. It is characterized by a granular or "salt-and-pepper" appearance, with small bright or dark spots scattered throughout the image.

<p align="center">
  <img 
    width="544"
    height="308"
    src="https://user-images.githubusercontent.com/101938119/213531959-61496e6f-abce-4d1b-a72d-9b925ac867f8.png"
  >
</p>

> Our goal here is to get from an image that looks like the one on the right.

It is caused by the constructive and destructive interference of light waves scattered by small, randomly distributed scatterers within the imaging system. The scattered waves combine to create a speckled pattern on the image. There can be different variations of noise disturbances such as multiplicative noise and additive nosie.

Speckle noise can be a significant problem in certain imaging applications, as it can reduce image quality, make it difficult to detect small features, and limit the ability to extract useful information from the image. Getting rid of it can be difficult but can prove to be very useful. As the thesis mentions, it can "[lead] to more accurate diagnosis and treatment of the disease".


## Graphical Depiction

In this case, we are talking about images and images having speckle noise, but I would first like to show speckle noise's effect on signals. Since images can be represented as vectors, this would be a good way to visualize what is happening. Since I am going to be converting the image we are working with to patches and then later to a vector, it is good to see what changes happen to a single vector. 

The noise formula is modeled by the following: $$\textbf{y} = AX_o\textbf{w} + \textbf{z}$$

Here is what all the variables are: 
- $\textbf{y}$ : This is the final measurement of the signal we end up with, and is the one that we see. 
- $A$ : This is a multiplicative constant (can be in the form of a matrix).
- $X_o$ : This is the original signal in the form of a matrix. The signal elements are on the diagonal of a square matrix.
- $\textbf{w}$ : The speckle noise, this is the main issue we are dealing with. 
- $\textbf{z}$ : This is the white gaussian additive noise. 

### Test vector
We can generate random values for speckle noise, additive white gaussian noise, and the original signal to see a sample comparison between $X_o$ and $y$. 

We can take $w$ as some random multiplicative values, ranging between 0.8-1.2 because we don't want too much difference, but just enough to see. 
We can take $X_o$ as the matrix of the array [5.4, 7.65, 9.4, 3.4] as spread along its diagonal. This was randomly generated.
We can take $z$ as a guassian random distributed noise:
Ignoring $A$ for now, we can just set it equal to 1.

Here, I am converting all of the lists to python vectors so that we can do operations on them.

The way to do this in python is the following: 

- Let's assume that the randomly generated vector is a column vector (5x1). 

```
import numpy as np

# Generate a 5x1 vector (this can be our Xo - original vector) 
x = np.random.rand(5, 1)
print("Original Vector:")
print(x)

# Generate multiplicative constant in the form of a matrix (must be 5x5 because of our conditions) with random values between 0.1 and 1.9
A_matrix = np.random.uniform(low=0.1, high=1.9, size=(5, 5))
x = x * A_matrix
print("Vector with multiplicative constant included")
print(x)


# Generate multiplicative noise (this is the speckle part)
multiplicative_noise = 0.5
x = x * (1 + multiplicative_noise * np.random.randn(5, 1))
print("Vector with multiplicative noise:")
print(x)

# Generate additive noise
additive_noise = 0.1
x = x + additive_noise * np.random.randn(5, 1)
print("Vector with additive noise:")
print(x)
```

## Test Images
As aforementioned, I used 4 very high quality images of lungs with pneumonia. Here is one of them - they all look pretty similar from a third person perspective. 

<p align="center">
  <img 
    src="https://github.com/akhilvreddy/ECE491-SpecialProblems/blob/main/Training%20Images/im2_pn_normal.jpeg"
  >
</p>

If you look closely, the image has a black border on all sides. Since each pixel can inlfuence the model, we would like to remove the border. This is what I did to remove the border from the image: 

``` 
import cv2
# Load the image
image = cv2.imread("image.jpg")

# Define the border color (black)
lower = [0, 0, 0]
upper = [0, 0, 0]

# Create a mask for the border color
mask = cv2.inRange(image, lower, upper)

# Remove the border color
image_without_border = cv2.bitwise_not(image, image, mask=mask)
```

Another way to do this is using the properies of python. 

``` 

```

### Image Patches

Before moving further in the program, we would need to convert the testing images that I have into smaller patches. There are two reasons that we would have to do this. 

- **Computational Power.** The main reason to change to patches is because the machines we are using cannot handle/support that much computation. Even after running our code with many patches from a singular high quality image, it took my machine (lenovo with i5) about 17.5 minutes to run the autoencoder model. Having to run multiple of models for multiple high quality images would be too much. GPU would be needed for this case. 

- **Efficiency.** Making patches like this is decreasing computational power for a similar accuracy. We would want to do this if we are working without funding / low budget. 

### Test patches
Given an image like the one above, we can get *n* number of patches from that specific image using the *sklearn* library. 

```
from sklearn.feature_extraction import image

# Load the image
image = ...

# Define the patch size (e.g. (64, 64))
patch_size = (64, 64)

# Extract the patches
patches = image.extract_patches_2d(image, patch_size)

# convert from 4d array down to 3d array for better use
patches = patches.reshape(-1, patch_size[0], patch_size[1])

print(patches)
```

> imported our test images into this code segment
> 
> [[[174 201 231],[174 201 231]],[[173 200 230],[173 200 230]]] is the output I got after running the code

If we wanted to go the other way around (patches back to the image), *sklearn* has a library to do that as well. 

```
from sklearn.feature_extraction import image

# Load the patches
patches = ...

# Reconstruct the image
image = image.reconstruct_from_patches_2d(patches, (height, width))
```

## Tackling the problem

### Autoencoders 
Autoencoders can be used to remove the noise and distortions from the images by training the network to reconstruct the original, noise-free image from the noisy input image. Autoencoders don't have a specific defintion but they are capable of reducing the data dimensions by ignoring noise in the data. It will then expand the data out agian to the dimensions of the initial dataset. There are usually four components inside an autoencoder, all of these combined make it up.
- Encoder  
  * In which the model learns how to reduce the input dimensions and compress the input data into an encoded representation.
- Bottleneck 
   * The layer that contains the compressed representation of the input data. This is the lowest possible dimensions of the input data.
- Decoder
  * In which the model learns how to reconstruct the data from the encoded representation to be as close to the original input as possible.
- Reconstruction Loss
  * the method that measures measure how well the decoder is performing and how close the output is to the original input.

Here is an image depicting the way autoencoders work. This image shows a one-layer design, but a lot of autoencoders can have many more than a single layer.

<p align="center">
  <img 
    width="464"
    height="328"
    src="https://github.com/akhilvreddy/ECE491/blob/main/ourimage.png"
  >
</p>

Autoencoders are the biggest tools that allow us to solve inverse problems. The way we are going to solve the vector equation is by trying to inverse it, kind of like an algebraic equation, but we cannot do the same elementary operations for a vector equation involving matrices. 

### Recovery Algorithms

One of the ways to get the signal (or image in this case) back from speckle noise is by Projected Gradient Descent. The cost function in this case would be 

#### Recovery using Generative Functions (GFs)

Before getting into other more complicated algorithms, I want to go over others. Recovery using GFs is used for 


#### Projected Gradient Descent (PGD)

This is one of the ways we reduce the cost function step-by-step to drive closer to the solution everytime. Understading the pseudo-code is very important before coding it up. 

* We are assuming that the result is **x<sub>t</sub>**.

```
Xo = diag(xo)    //initalize the matrix
Bo = A(Xo)^2(A^T)

for t = 1:T do 
  for i = 1:n do
  
  s(t, i) = *some algorithmic step*
  
  end 
  
  x_t = pi*c_r*s_t
  Xt = diag(xt)
  Bt = A(X_t)^2(A^T)
end
```

Let's unpack this alogrithm. The first two lines should be pretty self-explanatory - we are just setting up the basic matrix and constants in order to do the first iteration calcuations.

The nested for loops is where we get into the meat of the algorithm. For the inner-most loop, we are trying to find values of s_(t,i). Since t stays the same for a single loop, we get n specific values of the s vector. 

After exiting that loop, we set all the values of the x_t vector to a specific constant times those values. 

The X_t matrix gets updated from this everytime. 

By running through both of these for-loops we are inching closer towards an answer everytime. The main line that is getting us to reduce our cost-function is: 

<p align="center">
  <img 
    width="508"
    height="48"
    src="https://github.com/akhilvreddy/ECE491/blob/main/pic2.png"
  >
</p>

## Starting with the Autoencoder class

### Dependencies 
As you can see in the main jupyter notebook, we have a couple of dependencies for this program. 
```
from sklearn.datasets import load_sample_image
from sklearn.feature_extraction import image
import numpy as np
from matplotlib import pyplot as plt
import cv2
```

- Scikit-learn (sklearn) is a popular Python library for machine learning. It offers a wide range of tools for tasks such as classification, regression, clustering, and model selection, and it is built on top of other popular Python libraries such as NumPy and Matplotlib. It provides a consistent interface to various machine learning models, making it easy to switch between them and to perform common tasks such as feature extraction, model selection, and evaluation. I have used it through the program, especially in my autoencoder class. 
- NumPy is a Python library that provides support for large multi-dimensional arrays and matrices of numerical data, as well as a large collection of mathematical functions to operate on these arrays. It is a fundamental package for scientific computing with Python. 
- OpenCV (Open Source Computer Vision Library) is a library of programming functions mainly aimed at real-time computer vision. It provides a number of features such as image processing, video analysis, object detection, and machine learning.
- Matplotlib is a plotting library for the Python programming language and its numerical mathematics extension NumPy. It provides an object-oriented API for embedding plots into applications using general-purpose GUI toolkits like Tkinter, wxPython, Qt, or GTK. It provides a high-level interface for drawing attractive and informative statistical graphics.

All of these libraries together help us get to our final goal - a fully working autoencoder method which stores the *MSE (mean squared error)*. 

### Putting our Images into Python
As aforementioned, we start by importing our image into python from our desktop. We need to divide by 255 because we want to normalize it. After normalization, we output the image and as you can see, it looks pretty much the same as what we had above. Thsi is because if you divide every pixel value by the same value, you are going to get the same image, just scaled down in value. 
```
# inputting the image from 
input_img = "im1_pn_normal.jpeg"

#saving the images that we have into vector variables
img = cv2.imread(input_img,0)

# the following command will help us understand what the image will look like (vectorized)
img = img/255
# this is going to show us the dimensions of the image  (we can make adjustments based off this)
```

We can now output this image using the following command:
```
plt.imshow(img,cmap='gray')
```
