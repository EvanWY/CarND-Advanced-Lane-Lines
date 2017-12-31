## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

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

[chess]: ./output_images/chess.png ""
[bird]: ./output_images/bird.png ""
[fit]: ./output_images/fit.png ""
[fit2]: ./output_images/fit2.png ""
[threshold]: ./output_images/threshold.png ""
[undistorted]: ./output_images/undistorted.png ""
[plot]: ./output_images/plot.png ""
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./advance_lane_lines.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][chess]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][undistorted]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color (HLS space) and gradient thresholds to generate a binary image (thresholding steps at part 3 of the notebook).  Here's an example of my output for this step. 

![alt text][threshold]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `birdsEyeViewWarp()`, which appears in part 4 of the IPython notebook).  The `birdsEyeViewWarp()` function takes as inputs an image (`img`). I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32([[-80,690],[1420,690],[555,450], [730, 450]])
dst = np.float32([[0, 720], [1280, 720], [0, 0], [1280, 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| -80, 690      | 0, 720        | 
| 1420, 690      | 1280, 720      |
| 555, 450     | 0, 0      |
| 730, 450      | 1280, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][bird]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

First I use the sliding window method to find out all the pixels that are included in all the boxes as shown in this image (part 5 in the ipython notebook)

![alt text][fit]

Then I seperate pixels into left and right, and fit their position with a polynominal.

![alt text][fit2]

After the first frame, I use the polynominal from last frame, and filter the points that are within a certain distance of the previous curve.

At each frame, I use polynominal parameters from last frame to do a low pass filting

```python
new_left_fit = np.polyfit(lefty, leftx, 2)
new_right_fit = np.polyfit(righty, rightx, 2)
f = 0.8
laneInfo.left_fit = laneInfo.left_fit * f + new_left_fit * (1-f)
laneInfo.right_fit = laneInfo.right_fit * f + new_right_fit * (1-f)
```

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines 117 through 118 in my code in the notebook in the function `findLane()`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in part 7 in the notebook in the function `overlayImg()`.  Here is an example of my result on a test image:

![alt text][plot]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_optimized_project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

Camera calibration, bird view warping are relatively easy.

Finding out the appropriate parameter for the threshold image is very tricky. Different lighting condition, camera angle, road shape will all affect the result in this part. This is also the part that I think the pipeline will likely fail for difficult testcase. I think the best way to solve this is to test on more data.

Fitting line rely on the threshold image. If the threshold image is good enough, the fitting will end up very well.
