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

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of the IPython notebook located in ./AdavancedLaneFinding.ipynb 

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test images using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, below is an example of the original image and the undistorted image.

Original Image:

![alt text][image2]

Undistorted Images:

![alt text][image2]
A better visualization can be provided with the chess board images as the in undistortion in the chess board images is readily visible.

Original Chess Board Images:

![alt text][image2]

Un-distorted Chess Board Images:

![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to obtain the final binary output images. 
First I have done the color thresholding on the warped images. For the color thresholding I have first extracted the R channel of the image. Then I have converted the image to the HLS color space so that it is easier to identify the lane lines robustly in using the S channel. Further I have used a combination of The H and the L channels as well. I have used the a Hue mask for the Yellow lane Lines and Combined it with a lignteness channel so as to take care of the color variation in the lane lines due to varying brightness. The code for the color thresholding can be found in this "ColorThresholding" function. Below is the output after combining all the color thresholds.

![alt text][image3]

Next I have performed the gradient thresholding on the warped images. For the gradient threshold I have used the magnitude of gradient as that seems to give the best result for me. I had also tried using just Sobelx but I did not seem to give good results as there was a lot of noise in the result. The code for the gradient threshold can be found under the function "GradThresholding" in the notebook. Below is the visualization of the result I have obtained after gradient thresholding on the warped images.

![alt text][image3]

Finally I combined the result that I obtained from the color and the gradient thresholding. Below is the result that I obtained after cobining the color and the gradient thresholds.

 ![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The first step of the processing pipeline that I have performed is the Perspective transform. I have performed the perspective transform so as to have a "Birds eye view" of the lane lines so as detect the lane lines better with their curvature.
Below are the source and destination points that I have taken for the perspective transform. I have chosen the source and destination points manually and have arrived at the final result after tweeking them for the best result.

The resultant source and destination points are:
Here the width and height are the width and height of the image.

|Source        | Destination   | 
|:-------------:|:-------------:| 
| 600, 450      | 250, 0        | 
| 700, 450      | width-390, 0      |
| 280, 710     | 250, height      |
| 1080, 710      | width-390, height        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.
Below is the result of the warped images:

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The next thing aim I had was to draw the lane line properly on the image. For that I plotted a histogram for the final resultant images obtained after combining the gradient and color thresholding.
Below is a visualization of a histogram that I obtained for one of the test images. 

![alt text][image5]

The I used the sliding window technique to find the lane lines and fit it to a second degree polynomial so as to obtain a smooth line for the lane.
Below is the result that I obtained after the sliding window technique. 

![alt text][image5]
 
#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

To plot back the result onto the original image I have unwarped the image by using minverse and then combined it back with the original image.
Below is the final result that shows the lane lines marked on the test images.
 
![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

while working on the project some of the problems that I faced were for finding the correct set of source and destination points to 
