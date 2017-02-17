
**Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./output_images/distort_image.jpg "distorted chessboard Image"
[image2]: ./output_images/undistort_image.jpg "Undistorted Chessboard Image"
[image3]: ./output_images/undistort_road_image.jpg "Undistorted Image of the road"
[image4]: ./output_images/H_image.jpg "H image"
[image5]: ./output_images/S_image.jpg "S image"
[image6]: ./output_images/L_image.jpg "L image"
[image7]: ./output_images/B_image.jpg "B image"
[image8]: ./output_images/Mag_image.jpg "gradient magnitude image"
[image9]: ./output_images/comb_image.jpg "combination image"
[image10]: ./output_images/Road_image_with_points.jpg "Road image with source points"
[image11]: ./output_images/pers_road_image.jpg "perspective transformed road image"
[image12]: ./output_images/poly_fit_image.jpg "Image with lane lines marked"
[image13]: ./output_images/warped_back_image.jpg "warped back image to the road"
[video1]: ./result.mp4 "Video"

###Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of `P4.ipynb`.  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

### Original Image

![alt text][image1]

### Undistorted Image

![alt text][Image2]

###Pipeline (single images)

####1. Provide an example of a distortion-corrected image.

#### Undistorted Image of the road

I applied the same technique I used to undistort the chess board image to the road image like this:

![alt text][image3]

####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
I used a combination of color and gradient thresholds to generate a binary image. I tested with different combinations of color and gradient threshold and I finally came up with a combination of l channel in HLS color space and b channel in Lab color space.  Here's the procedure of how I came up with this combination.

I tried with H channel in HLS to help me find the lines. I used a threshold channel of [17, 30]. In this channel H is able to detect the yellow line perfectly in all conditions. But It is associated with a large noise. The H channel image is as shown below.

![alt text][image4]

I used S channel in a threshold range [100, 255]. S is able to detect all color lines but I found it is not perfect in all conditions. Then I thought of using S channel with h channel which produced very good results. The S channel is a s shown below.

![alt text][image4]

I used L channel in a threshold range [200, 255]. L channel has a capability to detect white lines nearly perfect in all conditions. The L channel image is as shown below.

![alt text][image6]

I also tried with B channel in Lab color space in a threshold range [155,255]. B produced good results finding the yellow line very perfectly in all conditions. This made me use B channel in all my test combinations to find the lane lines. The image of B channel is as shown below.

![alt text][image7]

I tried using Magnitude gradient threshold with range [50,200]. But color spaces are lot more better in detecting the lane lines. I thought of just using the color space would prodice good results in my final result. The image produced by gradient thresholding is as follows.

![alt text][image8]

After trying different combinations like ((S&H) | L), (H | L), (S | L), (B | L), (S | B), ((S&H) | L | B), I got good results with (L | B ) and ((S&H) | L | B). I used (L | B) combination to finally produce my result. The combination image is shown below:

![alt text][image9]

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `perspective_transform()`, which appears in code cell 3 in `P4.ipynb`. The `perspective_transform()` function takes as inputs an image (`img`).  I chose the hardcode the source and destination points in the following manner:

```
src = np.float32([[545,470],[750, 470],
                      [60, 720],[1280, 720]])
dst = np.float32([[0, 0], [1280, 0], 
                     [0, 720],[1280, 720]])

```
This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 545, 470      | 0, 0        | 
| 60, 720      | 0, 720      |
| 1280, 720     | 1280, 720      |
| 750, 470      | 1280, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

### Points selected for Warped Image

![alt text][image10]

### Perspective transformed image

![alt text][image11]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I  have computed a histogram and found peaks in left and right parts of image and formed a rectangle window around those peaks and then stack those windows which follow the line by checking and if the number of points in each window is greater than 50 then change the x position of the window to the mean x position of the points. I maintain record of all the points in the window and then used polyfit() functions to fit a curve to those points.

Once I fit a curve in a frame of video, for the next frame I do not repeat the whole procedure but search within the region around the pervious fitted curve

![alt text][image12]

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I calculated the radius by the following code which takes care of the conversion from pixels to meters which is implemented in code cell 12 in my code in `P4.ipynb`

```
ym_per_pix = 30/720 # meters per pixel in y dimension
xm_per_pix = 3.7/700 # meters per pixel in x dimension

left_fit_cr = np.polyfit(lefty*ym_per_pix, leftx*xm_per_pix, 2)
right_fit_cr = np.polyfit(righty*ym_per_pix, rightx*xm_per_pix, 2)
left_curverad = ((1 + (2*left_fit_cr[0]*y_eval*ym_per_pix + left_fit_cr[1])**2)**1.5) / np.absolute(2*left_fit_cr[0])
right_curverad = ((1 + (2*right_fit_cr[0]*y_eval*ym_per_pix + right_fit_cr[1])**2)**1.5) / np.absolute(2*right_fit_cr[0])

```


####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in my code in code cell 13 in `P4.ipynb`.  Here is an example of my result on a test image:

![alt text][image13]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./result.mp4)

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The code is fairly robust to different light conditions. But it has problems detecting the lines in shadows and road with bright color where it has problem differentiating the lines with the road.

I think estimating the road according to the full width of the road and previous measurement of the lane lines will come in handy some times. There should be some trade of with the estimation and the actual measurement. In this way I guess, the system will be more robust to different road conditions and light conditions. 

