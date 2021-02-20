# Self-Driving Car Engineer Nanodegree Program

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

[image1]: ./camera_cal/corner_calibration10.jpg "Chessboard Calibration"
[image2]: ./camera_cal/test.png "Undistorted Chessboard"
[image3]: ./test_images/test3.jpg "Example Image"
[image4]: ./output_images/test3_undist.jpg "Undistort Example"
[image5]: ./output_images/test3_binary.jpg "Binary Example"
[image6]: ./output_images/test3_binary_warped.jpg "Perspective Transform Example"
[image7]: ./output_images/test3_fitlane.jpg "Fit Visual Example"
[image8]: ./output_images/test3_binary_result.jpg "Fit from Previous Example"
[image9]: ./output_images/test3_output.jpg "Output"
[video1]: ./project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the IPython notebook located in "./camera_cal/camera_cal.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image by using `cv2.drawChessBoardCorners`.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection. The detected corners are drawn on the chessboaord iamges as shown below:

![alt text][image1]

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function. Note that the obtained coefficients are saved in a pickle file "./camera_cal/cal_picle.p".

In the first cell of the IPython notebook located in "./P2.ipynb", I loaded this picle file. Then in the sixth cell, I applied this distortion correction to the test images using the `cv2.undistort()` function and the examplary result is shown below: 

![alt text][image2]

### Pipeline (single images)

To demonstrate the pipeline, I will use following test image:

![alt text][image3]


#### 1. Provide an example of a distortion-corrected image.

First, I apply the `cv2.undistort` function and the result is shown below:

![alt text][image4]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

Different thresholding functions are defined in the second cell of the IPython notebook located in "./P2.ipynb". Then in the pipeline for of test images (sixth cell),  used a combination of color and gradient thresholds to generate a binary image. Here's an example of my output for this step.

![alt text][image5]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform is in the sixth cell of IPython notebook located in "./P2.ipynb". The transformation and inverse transformation matrices are calculated by using `cv2.getPerspectiveTransform` function and then transformation is performed using `cv2.warpPerspective` function as shown below:

```python
M = cv2.getPerspectiveTransform(src, dst)
Minv = cv2.getPerspectiveTransform(dst, src)
warped = cv2.warpPerspective(combined_binary, M, (xsize, ysize), flags=cv2.INTER_LINEAR)
```

The `cv2.getPerspectiveTransform` function takes source (`src`) and destination (`dst`) points as inputs.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32([[xsize*0.46,ysize*0.62], [xsize*0.15,ysize], [xsize*0.88,ysize], [xsize*0.54,ysize*0.62]])
offset = 300
dst = np.float32([[offset, 0], [offset, ysize], [xsize-offset, ysize], [xsize-offset, 0]])
```

I verified that my perspective transform was working as expected i.e. the the lines appear almost parallel in the binary warped image as shown below:

![alt text][image6]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The functions "find_lane_pixels()" and "search_around_poly()" are defined in the third cell of IPython notebook located in "./P2.ipynb" to determine lane line pixels. 

The "find_lane_pixels()" function uses histogram of the binary warped image which is calculated by using the function "hist()" to determine the starting points of the lane lines. After determining the starting points of the lane lines as the two highest peaks of the histogram, the sliding window technique is used to determine where the lane lines go. After finding the all pixels belonging to the each lane line, a second order polynomial is fitted to represent lanes using function "fit_polynomial()" in the third cell of IPython notebook located in "./P2.ipynb". The following image is shown the result of the described procedure:

![alt text][image7]

On the oter hand, the "search_around_poly()" function is using previous detection information instead of a blind search to detect lane line pixels. If we have detected the lanes in the previous frame of the video, we can just search in a margin around the previous lane line position, like in the below image:

![alt text][image8]

The green shaded area shows where we searched for the lines this time. So, once you know where the lines are in one frame of video, you can do a highly targeted search for them in the next frame.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I defined two different functions "measure_curvature_real()" and "car_offset()" in the fourth cell of IPython notebook located in "./P2.ipynb" to calculate radius of curvature of the lane and the position of the vehicle with respect to center.


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I defined the function "draw_lane_with metrics()" to visualize the result with the radius of curvature of the each lane and the position of the vehicle with respect to center in the fifth cell of IPython notebook located in "./P2.ipynb".  Here is an example of my result on a test image:

![alt text][image9]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

To track the lane lines in a video, the class lane is defined in the 7th cell of IPython notebook located in "./P2.ipynb" and the extended version of the described pipeline is implemented in the function "process_iamge()" in the 8th cell of IPython notebook located in "./P2.ipynb". In this function, sanity check is performed to track lane lines. In order to determine the fitted line is a correct representation of the lane lines or not different properties of the lines are checked and the robustness of the pipeline is tried to increase by this sanity check. Also, the lane lines are showing with the smoothing function using the average of last `n` found lanes to obtain a cleaner result.

Although the designed pipeline is properly worked for project video, the performance of the pipeline is decreased for the more challenging videos. The main problem is originated from creating a proper thresholded binary image when the lighting conditions are more challenging. Therefore, I need to find a more robust thresholding functions to improve the performance of the pipeline.
