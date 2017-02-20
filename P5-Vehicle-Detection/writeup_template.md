
**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./output_images/car.png
[image2]: ./output_images/car_hog.jpg
[image3]: ./output_images/notcar.png
[image4]: ./output_images/notcar_hog.jpg
[image5]: ./output_images/cary.jpg
[image6]: ./output_images/hogy.jpg
[image7]: ./output_images/bbox.jpg
[image8]: ./output_images/all_boxes_car.jpg
[image9]: ./output_images/heat_map.jpg
[image10]: ./output_images/all_boxes_car.jpg
[image11]: ./output_images/final_car_boxed.jpg

[video1]: ./project5_result.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

###Histogram of Oriented Gradients (HOG)

####1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the code cell 3 of `P5.ipynb`.  

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes and their HOG visualizations:

### Car

![alt text][image1]

###HOG visualization of car

![alt text][image2]

### Not a Car

![alt text][image3]

### HOG visualization of a non car

![alt text][image4]

The `extract_features` function is called for the list of images,  in the code cell 5 in `P5.ipynb`. It extracts Hog features, spatial and color histogram features. However, I have only used HOG features of each color channel of a color space. The results of these extracted features for each set of images are then stacked and normalized (using sklearn's `StandardScaler` method) and then split into training and testing datasets (using sklearn's `train_test_split` method).

I then explored different color spaces like HLS, HSV, YUV, RGB. But I achieved highest accuracy with YUV color space. So I used YUV for my feature extraction. 

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I have built a classifier and tried to detect cars using these paramters in an image. The following parameters of HOG in YUV color space are the best set of paramters that worked well on the test images in detecting cars.

```
color space = 'YUV'
orientations=8
pixels_per_cell=(8, 8)
cells_per_block=(2, 2)
```

#### Y image of car in YUV space:

![alt text][image5]

#### HOG visualization of Y image of car in YUV space:

![alt text][image6]

####2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters and I finally chose the parameters shown above. I decided my final parameters based on length of the vector it produced as it directly reflects the time consumption of the code and accuracy I am getting with my classifier. I changed the number of orientations from 5 to 10 and 8 orientations worked best for me.

####3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I have only used HOG features and not included color or spatial features to avoid very huge feature vectors. Extracting features and training the classifier is implemented in code cell 8 in `P5.ipynb`. I have divided the data(car and non car images) into training and testing datasets and then extracted features (HOG) using `extract features` function. I have then scaled and normalized the features to zero mean and unit variance and fed to Support vector machine classifier with appropriate labels(car or not car). The model learnt by the classifier was tested to predict labels on the testing dataset. I achieved an accuracy score of 98.65% on the testing data set.

###Sliding Window Search

####1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

This is implemented in code cell 14 in `P5.ipynb` where I have a function for sliding the window and searching for vehicles in this window. I have only searched for cars in the lower half of the image which is mainly the roads. The sliding window overlaps for 0.75 (both x and y).These values are chosen by experimenting different combinations and the ones which are fast and have obtained good detections have been chosen. The classifier checks if there is a car in the frame and all windows where a vehicle is predicted is returned. The cars which are nearer appear bigger and which are further appear smaller. So I have restricted the larger windows to lower part of the search area and I have also searched on three scales of window sizes small, medium and large in order to be able to  detect cars in all positions in an image which are found to be best on the test images. The following are the window sizes in x and y directions I have used which also in code cell 14 in `P5.ipynb`

```
75,75 
100,100
125,125
```
Here is the image containing the list of windows that are usually searched for:

![alt text][image7]

####2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on three scales using YUV 3-channel HOG features. The classifier predicted whether a given window has a car with some confidence. The windows that returned high confidence are selected to be robust to false positives and then I created a heatmap which is similar to probability of detecting a car in a window. If there are multiple windows in a region then the heatmap has high value and indicates higher probability of detecting a car which can be done by thresholding heatmap after trying different values for a threshold and the best threshold for which I could reasonably detect cars and eliminate false positives is 2.


The following image shows all the windows returned by the `search_windows` where the car is detected.


![alt text][image8]


---

### Video Implementation

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project5_result.mp4)


####2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

False positive may occur when a window has no car and was detected to be a car. One of the simplest way to eliminate the False positives is using multi-frame accumulated heatmap  where I stored the last 15 heatmaps and calculated the average heatmap and then thresholded the heatmap. When you consider multiple frames the chances of False positive getting detected multiple times is tough and choosing a good threshold on the average heatmap can eliminate the False positives to a very good extent. This is implemented in code cell ## in `Vehicle_detect function`  
To combine the result of overlapping windows I have used connectedness property of pixels in a heat map to combine all the combine the overlapping windows using `scipy.ndimage.measurements.label()`. The following are the results obtained:

#### Heatmap

![alt text][image9]

#### All the windows corresponding to heat map

![alt text][image10]

#### Combined window using scipy.ndimage.measurements.label() by using 8-connected

![alt text][image11]

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I feel there should be a better heuristic for selecting parameters like the colorspace and hyper parameters of HOG  which were experimented by checking their performance on the test images. The pipeline is likely to fail in case of occlusions and also if there are False positives between true positives. Also I found that it is taking a lot of time to compute for a 50 second video. I think if this algorithm have to work in real time, faster GPUs might be necessary. To make it more robust Bag of words type of approaches could be used to handle occlusions.
