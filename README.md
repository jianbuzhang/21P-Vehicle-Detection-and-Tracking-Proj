# 21P.Vehicle-Detection-and-Tracking-Proj
### Bob Jin,2018/06/01

**Introduction:**This repository contains a computer vision solution for vehicle detection and tracking in a self driving car.

---
## The goals/steps of this project are the following:
* Training a classifier.
* Liniear SVM.
* Searching within a frame.
* Combine the car detection with lane line detection together
* Reflections

[//]: # (Image Reference)

[img1]: ./saved_figures/car.png "car1"
[img2]: ./saved_figures/color_hist.png "color_hist"
[img3]: ./saved_figures/spatial.png "spatial"
[img4]: ./saved_figures/hog_comparison.png "hog_comparison"
[img5]: ./saved_figures/hog_subsampling.png "hog_subsampling"
[img6]: ./saved_figures/vehicle_detect_tracking.png "vehicle_detect_tracking"


---
## Writeup/README
### 1. Training a Classifier
This project uses training images from a combination of the [GIT vehicle image database](http://www.gti.ssr.upm.es/data/Vehicle_database.html) and the [KITTI vision benchmark suite](http://www.cvlibs.net/datasets/kitti/).You can download the data sets for [vehicles](https://s3.amazonaws.com/udacity-sdc/Vehicle_Tracking/vehicles.zip) and [non-vehicles](https://s3.amazonaws.com/udacity-sdc/Vehicle_Tracking/non-vehicles.zip)if you want to reimplement this project.

![alt text][img1]

A convolutional neural network could learn to detect cars like this pretty easily in an image. It does that by breaking an image down into small convolutions which generate edges.The edges ensemble into shapes and so on. We need to transform the image into a more useful feature space for the reason that we are going to use traditional machine learning algorithms. So we will use 3 different feature extraction methods.
It comes first that we convert every image to **the YCrCb color space**, *Y* stands for luminance,*C* stands for red-difference chroma component, and *Cb* is the blue-difference chroma component. YCrCb is perfect in making a distinction between surfaces under different environment of light and color.

#### Histogram of color
This is exactly what you think! Just tally up the values for each color channel across the X-axis. This is useful thinking about a car, but it can lead to false positives if it is the only feature.

![alt text][img2]

#### Spatially Binned Colors
Another quick and easy feature vector, spatially binned color takes an image, resizes it to a smaller size, then unravel it row by row into a one-dimensional vector. The image loses resolution and segments of the image with similar X values in the image will have the same pixel value in the feature vector.

![alt text][img3]

#### Histogram of oriented Gradients
HOG is an mazing algorithm, please read the instruction by clicking [here](http://lear.inrialpes.fr/people/triggs/pubs/Dalal-cvpr05.pdf) or the [video](https://www.youtube.com/watch?v=7S5qXET179I) by its creator. In a nutshell, HOG takes an image and breaks its down into a grid. Then, in each cell of the grid, every pixel has its gradient direction and magnitude computed. Those gradients are then but into a histogram with a number of bins for possible directions, however gradients with larger magnitudes contribute more than smaller magnitudes. The cell is then assigned the overall gradient direction and magnitude. This algorithm produces a feature vector that can be thought of like a signature for different objects. This is because the grid format helps to preserve structure - it provides a strong signal while reducing noise. If every gradient direction and magnitude were used as input, the signal would be drowned out.

![alt text][img4]

It is amazing that each cell of the HOG looks similar to the edges created by the levels of a convolutional neural network! In my project I binned the HOG into **10 directions** and each cell consisted of **eigth pixels**. I used the HOG features from **each color channel** as well.

#### Linear SVM
Since we have a feature vector for image, we can train a classifier on the training data. I used a **LinearSVM** because it is efficient for classification, so it runs very fast. Before we pass the feature vector to the classifier, it should be normalized or standardized.
Finally I accomplish a validation accuracy of **98.99%** and it took **20.05** seconds to train the model.

---
### 2. Search within a frame
The following step is to identify cars within a frame of dash cam footages by using the classifier. I used a **sliding search window** method for this. I started by removing the top and bottom of the frame, which contain the sky and the hood of the car. Next, I take the HOG feature for the **entire image** which is of great importance. Then, the image is split into search windows the same size as the input for the model (64x64 pixels) with a suitable amount of overlap. Then each search window pulls out the HOG feature information from the array it was stored in for the entire image, which is **much less expensive** than trying to do HOG extraction on every search window! Finally, the model predicts on each search window's feature vector.

![alt text][img5]

---

### 3. Combine the car detection with lane line detection together
In the file **2_Search and Classify**, the function `vehicleTrack` set the image as input, then add rectangle around the car,then give out the image with rectangle in it. Finally,according to the previous lane line detection project we have conducted, use the function `fl_image` to create the figures with both car detection and lane line detection in it.

![alt text][img6]

---
### 4.Reflection
The project is mainly a showcase of the power of being explicit. Often times we think of deep learning as a cure-all, but there are situations in which explicit computer vision is much better and traditional machine learning is much faster. This project has a fast backend, however the drawing of bounding boxes, radius is very slow. I can imagine sending information to a robotics in realtime by using a pipeline, but not for displaying a HUD to a person. Besides, this pipeline is not robust enough to handle the driving conditions that it needs to be useable or efficient:
  1. Rain/snow/etc
  2. Roads that have little lane markers
  3. Occlusion by vehicles/signs/etc