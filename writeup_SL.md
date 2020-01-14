# *Advanced Lane Finding*
## Writeup 
### Project 2. "Becoming a Self-Driving Car Engineer" Nanodegree (Udacity)

The learnings and material from the corresponding lectures were used in this project.

---

**Advanced Lane Finding Project**

The overall goals is to write a software pipeline to identify the lane boundaries in a video from a front-facing camera on a car.

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

[image1]: ./output_examples/UndistortedChessboard.png "Undistorted Chessboard"
[image2]: ./output_examples/Undistortedcar.png "Undistorted Car"
[image3]: ./output_examples/binary.png "Binary Example"
[image4]: ./output_examples/Warpedexample.png "Warped Example"
[image5]: ./output_examples/WarpedPoints_example.png "Warped Example with Points"
[image6]: ./output_examples/Histogram.png "Histogram"
[image7]: ./output_examples/Slidingwindow.png "Sliding Window"
[image8]: ./output_examples/SlidingWindow2.png "Sliding Window 2"
[image9]: ./output_examples/RadiusCurve.PNG "Radius Equation"
[image10]: ./output_examples/FinalPipeline.png "Final Example"

[video1]: ./output_examples/project_video_ouput.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

This writeup is the explantation response of my code to the rubric criteria (above).

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this is located in the 'Camera Calibration' section. There were 20 chessboard images taken at different angles  to calibrate the camera. Chessboard size is set to 9x6 for the project. The camera calibration is computed once, and applied to distort the frame of the road images. It is assumed for the road test images, that the camera is located in the centre at the front of the car and does not change location. 

The parameters (nx, ny) were set and the object points were prepared, assuming they are in the same coordinate plane. Arrays were created in the real world plane and image plane (objpoints, imgpoints respectively) to store the points from the images. Using the list of images, they were stepped through and searched for the corners of the chessboards using prebuilt functions. If the corners were found, the object and image points were appended and the corners were drawn. It is noted that the function correctly identified the internal corners of the chessboard, although some images provided only had only had nx = 9, ny = 5 shown in the frame, so the program ignored them. 

Using the objectpoints and image points arrays, the camera calibration and distortion coefficients were calculated, performed on the image and returned an undistorted image. See the result below for the chessboard image. 
![alt text][image1]


### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

See the result below for the original and undistorted image for the test1 road example. It may be hard to notice the distortion, but take note of the rear of the cars.
![alt text][image2]


#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

View the 'Color and gradient thresholds' section. 
A variety of color (RGB, HSV, HLS) and  sobel, magnitude and gradient thresholds were explored to see the outcomes of each effect indivudally. A combination of color and gradient thresholds was used to crate a binary image which strongly picked up the lane lines. See below for an example of the binary image from test1.
![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

View 'Perspective Transform' in the code. 

The function warp(img) computes and applies the perpective transform using the defined image size, source and destination points. Minv is also returned to be used to unwarp the images later.

The following source and destination points were used (and were hardcoded):

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 205, 720      |  205, 720     | 
| 1120, 720     | 1120, 720     |
| 745, 480      | 1120, 720     |
| 550, 480      | 205, 0        |
    
Warped example:
![alt text][image4]

I verified that my perspective transform was working as expected by overlaying the `src` and `dst` points and lines onto the test1 image and its corresponding warped image, where the lines were shown to be parallel in the warped image. 
![alt text][image5]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Refer to the 'Polynomial Fitting' section in my code.I went through similar steps as the lectures to find the lane line pixels. 

The warped binary image was used, as the pixels are 0 or 1. A histogram technique was used accross the bottom half of the image to determine the which pixels were lane pixels and belonged to the left or right lanes. It was asssumed that lane lines were most likely vertically orientated.. The peaks were used to indicate the x-positiion location of the base of the lane lines. 

![alt text][image6]

Next, a sliding window was placed around the centres of the left and right lanes to track the lines to the top of the frame. Here, the lane lines were split into their left and right hand sides. This iterates through the actiaved pixels to track the curvature. using the x and y pixel positions from the lane line pixels, a second order polynomial curve was used to fit to the line. 

![alt text][image7]

The sliding window is no longer required as the algorithm is adjusted to search in a margin around the previous lane position, this is more efficient as the lanes lines do not move much from frame to frame.

![alt text][image8]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Refer to the 'Polynomial Fitting' section in my code. The radius of curvature was first calculated in pixels in the measure_curvature_pixels()function and then calculated in m with the Refer to the measure_curvature_real() function.

Solving for f(y) because the lane lines in the warped image are near vertical and may have the same x for more than one y value. Provided in the lecture notes, the equation for radius of curvature is:
![alt text][image9]


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

Refer to the 'Final Pipeline' section. 

Below is an example of the test1 image.

![alt text][image10]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Output video can be found in my github repository here
https://github.com/SamaraLove/AdvancedLaneFinding/blob/master/output_examples/project_video_output.mp4

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The differnce in light is a contributor, and some color/gradient thresholds work better for these instances. The shadows on the road were sometimes mistaken for lane lines. A more active algorithm could be used to alternate the thresholding based on the images provided, perhaps this will be experimented later with a neural network training dataset. 

The polynomial fit is good for some road lanes, but won't be the best equation for all depending on if the cars are turning through steep corners or hills. 
