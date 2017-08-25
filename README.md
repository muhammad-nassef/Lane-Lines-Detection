# Udacity Self-Driving Car Engineer Nanodegree Program

---

## Advanced Lane Finding Project

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

[image1]: ./output_images/1-calibration.png "Calibration"
[image2]: ./output_images/2-distortion.png "Distorion1"
[image3]: ./output_images/2-distortion2.png "Distorion2"
[image4]: ./output_images/3-perspective_transform.png "Perspective Transform"
[image5]: ./output_images/4-combined_gradient_thresholding.png "Combined Gradient"
[image6]: ./output_images/4-RGB-SPACE.png "RGB"
[image7]: ./output_images/4-HLS-SPACE.png "HLS"
[image8]: ./output_images/4-HSV-SPACE.png "HSV"
[image9]: ./output_images/4-HLS-Sthreshold(120,255).png "S-Threshold"
[image10]: ./output_images/4-RGB-Rthreshold(230,255).png "R-Threshold"
[image11]: ./output_images/4-combined_grad_thresh_alone(30-100)(10-100(0-0.5).png
[image12]: ./output_images/4-sthreshold_only(80-255).png
[image13]: ./output_images/4-rthreshold-only(230-255).png
[image14]: ./output_images/4-combined_all.png
[image15]: ./output_images/5-sliding_window_line_fit.png
[image16]: ./output_images/5-line_fit_from_prev_frame_fit_data.png
[image17]: ./output_images/test5.jpg
[image18]: ./output_images/7-draw_lane.png
[image19]: ./output_images/7-draw_lane_data.png
[video1]: ./project_video_output.mp4 "Video"


## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first part of the IPython notebook located in "./examples/my project.ipynb" the part is addressed "1. Camera Calibration"
I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.

Here is an example of the chessboard images with the corners added to on it:
![alt text][image1]  

#### note: some of this images don't appear as the number of chessboard corners were not found

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function which returns the distortion coefficients and the transform matrix.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image2]

### Pipeline (single images)

#### 2. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image3]

Here you can notice the correction by looking on the bottom left of the image and how it is corrected

###### Note: the distortion correction can be found in the notebook under the part "2. Distortion Correction"


#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp_img()`, which appears in the notebook under the part addressed "3. Perspective Transform".  The `warp_img()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32([(590,450),
                  (710,450), 
                  (280,680), 
                  (1100,680)])

dst = np.float32([(450,0),
                  (w-450,0),
                  (450,h),
                  (w-450,h)])
```
where w and h are the width and height of the `img`


I verified that my perspective transform was working as expected by drawing the `warped_img` image against the `img` with the `src` points plotted on it:

![alt text][image4]


#### 4. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image. This can be found under the part `4. Gradients and Color Thresholding`

I applied many steps and tried many combinations to get a reliable output the steps are as following:

* implement methods for gradient thresholding `abs_sobel_thresh()` , `mag_sobel_thresh()`, `dir_sobel_thresh()` and `combined_grad()` which comines the three types of gradient thresholding (Absolute | (Magnitude & Direction)). This functions has been tested and calibrated to give the best output as following (Kernel_size =3, abs_thresh(30,100), mag_thresh(10,100), dir_thresh(0, 0.5)) 
###### note: I used `interact()` function to tune the thresholds in a better way 

![alt text][image5]


* I then Visualized several Color spaces to pick the best channels for robust lane detection here is a visualization of 3 color spaces (RGB, HLS, HSV) ![alt text][image6] ![alt text][image7] ![alt text][image8]
It  can be noticed that the S-channel in HLS Space detects the lines very well and as mentioned in the lectures it gives a reliable results under several conditions. The R-Channel of the RGB space also performs very well in white lines. I tried thresholding the V-Channel also but it didn't give good results

* As a start I begin to threshold the S-channel of HLS Space and I reached with (thresh (120, 255)) which has been modified afterwards to this result : ![alt text][image9]
* The R-channel also performs well on the white lines so I thresholded it to get the following binary image with (thresh (230,255)): ![alt text][image10]
* I tested each of the three Thresholding methods (S-Threshold, R-Threshold, Combined_grad-Threshold) on all the test alone to see the effect of it under different shadow and color conditions then decide the best fit. Here are the results of each of them :

1. Combined_grad Thresholding Alone:![alt text][image11] 
2. S-Channel Thresholding Alone![alt text][image12] 
3. R-Channel Thresholding Alone![alt text][image13]

* As can be noticed from the test results the S-Channel gives the lines well but when the tree shadow appeared very high noise occurred and this could be handled for sure by decreasing the min_threshold but on the other hand more points shall be missed. The R channel performs very well but with limited performance on the shadow. The gradient combination is good but a lot of points are not needed as result of noise `(R-Channel | (S-Channel & Combined_grad))`
* Here is the final result: ![alt text][image14]





#### 5. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I used the sliding window technique which make use of the histogram of the binary image then detects the peaks of the binary image afterwards the image is divided into two parts and the peak of each part represents the lane start position which shall be the start of the sliding window. Then after calculating all the sliding windows we can fit a polynomial for each lane line. The implementation of this method can be found in the function `sliding_window_line_fit()` Here is a result the Sliding window on all test images:

![alt text][image15]

This method is very expensive and can be optimized after the first time of calculation. In the next frame we could skip the sliding window and just search in a margin around the previous line positions The implementation of this method can be found in the function `line_fit_from_prev_frame()`. Here is the output:

![alt text][image16]


###### Note: This part is addressed in the notebook "5. Sliding Window"

#### 6. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I used the same technique mentioned in the lectures by calculating the radius of curvature from the first and second derivative of the polynomial function and convert the result from pixel space to xy space. In the same method I calculated also the distance to center. The implementation can be found in `calc_curve_rad_and_center_dist()` I reused here many code parts from the lectures and from forums also to verify I'm calculating it correctly.
I tested it on the following image and the results was (Radius of curvature for example: 222.293340952 m, 219.24099399 m
Distance from lane center for example: 0.149796694145 m)
which looks reasonable !
![alt text][image17]


###### Note: This part is addressed in the notebook "6. Radius of Curvature Calculation"

#### 7. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in function `draw_lane()` and this briefly uses MINV to inverse the Warping that happened in perspective transform and warp it back to the original image then overlays the resulted polygon to the original image and here is the output of the previous test image: 
![alt text][image18]

Then I added the data on the image using `draw_data()` function and the output was like this:
![alt text][image19]


###### Note: This part is addressed in the notebook "7. Warp detected lane on the original image with calculated data"
### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The main issues that I faced were in choosing the best Color space channels and gradient thresholding I had to calibrate and tune many times to reach this output

I haven't tried the pipeline on the challenging video which could rise more critical cases to be solved but In the current state it performs well under different conditions

The improvement that I thought about it quickly I could make the perspective transform more robust (not with fix values) but dynamically calculated depending on the case.
