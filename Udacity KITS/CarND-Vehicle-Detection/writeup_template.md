## Writeup Template
### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./examples/car_not_car.png
[image2.1]: ./output_images/sample_vehicle_HOG.jpg
[image2]: ./output_images/sample_vehicle.jpg
[image3.1]: ./output_images/sample_non_vehicle_hog.jpg
[image3]:  ./output_images/sample_non_vehicle.jpg
[image4]: ./output_images/test4.jpg
[image5]: ./output_images/test1.jpg
[image5.1]: ./output_images/test3.jpg
[image5.2]: ./output_images/test5.jpg
[image5.3]: ./output_images/test1_heatmap_thresholded.jpg
[image5.4]: ./output_images/test1_heatmap_grayscale.jpg
[image6]: ./output_images/test1.jpg
[image6.1]: ./output_images/test1_heatmap.jpg
[image6.2]: ./output_images/test2.jpg
[image6.3]: ./output_images/test2_heatmap.jpg
[image6.4]: ./output_images/test3.jpg
[image6.5]: ./output_images/test3_heatmap.jpg
[image6.6]: ./output_images/test4.jpg
[image6.7]: ./output_images/test4_heatmap.jpg
[image6.8]: ./output_images/test5.jpg
[image6.9]: ./output_images/test5_heatmap.jpg
[image6.10]: ./output_images/test6.jpg
[image6.11]: ./output_images/test6_heatmap.jpg
[image7]: ./output_images/test1_heatmap_grayscale.jpg
[image7.1]: ./output_images/test2_heatmap_grayscale.jpg
[image7.2]: ./output_images/test3_heatmap_grayscale.jpg
[image7.3]: ./output_images/test4_heatmap_grayscale.jpg
[image7.4]: ./output_images/test5_heatmap_grayscale.jpg
[image7.5]: ./output_images/test6_heatmap_grayscale.jpg
[image8]: ./output_images/test1_processed.jpg
[video1]: ./project_video_out_YUV.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the fourth, fifth, and sixth code cells of the IPython notebook called `Vehicle Detection Workspace.ipynb`.  

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

I then explored different color spaces - like RGB, YUV, HLS and LAB - and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here are examples of using the `RGB` color space and HOG parameters of `orientations=9`, `pixels_per_cell=(8, 8)` and `cells_per_block=(8, 8)`:


![alt text][image2]
![alt_text][image2.1]
![alt_text][image3]
![alt_text][image3.1]

#### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters and settled with the the same parameters as my exploration of HOG: 9 orientation values, 8 pixels per cell, 8 cells per block. I attempted to use 8 orientation values, 4 pixels per cell, and 15 cells per block, but once I got around to the test video I noticed that the rectangles were unreliable. They would flash on and off or on different sections of the vehicles, meanwhile my final values flashed substantially less, and remained in approximately the same locations

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM using sklearn.svm.LinearSVC(). This was a simple solution with 6 lines, and I used YUV transforms for the training and test data sets.

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I decided to search the image starting at the crest of the road (at a y value of 350) and down to roughly the hood of the car (y value of 650) and came up with images similar to this one:

![alt text][image4]

The rectangles are scaled at 1, 1.5, 2, 2.5, and 3. I figured I can find the farthest and closest cars, as well as that which is in between. The heatmap ensures that multiple scaled windows can select features on a car and be coupled to select a predicted vehicle with greater precision and accuracy.

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on YUV 3-channel HOG features which provided a usable result.  Here are some example images:

![alt text][image5]
![alt text][image5.1]
![alt text][image5.2]
---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./output_videos/project_video.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob with more than two positives corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected. 

Here's an example result showing the heatmap threshold for one of the test images:

![alt_text][image5.3]

and the grayscale image of the two vehicle heatmaps:

![alt_text][image5.4]


### Here are six frames and their corresponding heatmaps:

![alt text][image6]
![alt text][image6.1]
![alt text][image6.2]
![alt text][image6.3]
![alt text][image6.4]
![alt text][image6.5]
![alt text][image6.6]
![alt text][image6.7]
![alt text][image6.8]
![alt text][image6.9]
![alt text][image6.10]
![alt text][image6.11]


### Here is the output of `scipy.ndimage.measurements.label()` on the integrated heatmap from all six frames:
![alt text][image7]
![alt text][image7.1]
![alt text][image7.2]
![alt text][image7.3]
![alt text][image7.4]
![alt text][image7.5]

### Here the resulting bounding boxes are drawn onto the last frame in the series:
![alt text][image8]

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

My pipeline has issues when cars are lined up in the line of sight(LOS). If vehicles in separate lanes, are passing eachother, the pipeline takes time to realize that there are two cars in the frame. Instead a larger and larger box is drawn until the cars are almost separated again in the LOS. This doesn't currently break the system but could cause issues in the long run

