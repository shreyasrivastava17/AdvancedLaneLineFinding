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

[image1]: ./output_images/camera%20caliberation.png
[image2]: /output_images/color%20channel%20output.png
[image3]: ./output_images/combined%20grad%20color%20output.png
[image4]: ./output_images/final%20output%20.png
[image5]: ./output_images/gradient%20threshold%20output.png
[image6]: ./output_images/histogram.png
[image7]: ./output_images/test%20images.png
[image8]: ./output_images/sliding%20window%20output.png
[image9]: ./output_images/test%20output.png
[image10]: ./output_images/undistorted%20Chess.png
[image11]: ./output_images/undistorted%20images.png
[image12]: ./output_images/warped%20images.png
[video1]: ./output_video/project_video_output.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of the IPython notebook located in ./AdavancedLaneFinding.ipynb 

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function. Below are the images that I used to calibrate the camera

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, below is an example of the original image and the undistorted image.

Original Image(Test Images):

![alt text][image7]

Undistorted Images(Test Images):

![alt text][image11]

A better visualization can be provided with the chess board images as the in undistortion in the chess board images is readily visible.

Original Chess Board Images:

![alt text][image1]

Un-distorted Chess Board Images:

![alt text][image10]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to obtain the final binary output images. 
First I have done the color thresholding on the warped images. For the color thresholding I have first extracted the R channel of the image. Then I have converted the image to the HLS color space so that it is easier to identify the lane lines robustly in using the S channel. Further I have used a combination of The H and the L channels as well. I have used the a Hue mask for the Yellow lane Lines and Combined it with a lignteness channel so as to take care of the color variation in the lane lines due to varying brightness. The code for the color thresholding can be found in this "ColorThresholding" function. Below is the output after combining all the color thresholds.

![alt text][image2]

Next I have performed the gradient thresholding on the warped images. For the gradient threshold I have used the magnitude of gradient as that seems to give the best result for me. I had also tried using just Sobelx but I did not seem to give good results as there was a lot of noise in the result. The code for the gradient threshold can be found under the function "GradThresholding" in the notebook. Below is the visualization of the result I have obtained after gradient thresholding on the warped images.

![alt text][image5]

Finally I combined the result that I obtained from the color and the gradient thresholding. Below is the result that I obtained after cobining the color and the gradient thresholds.


 ![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The first step of the processing pipeline that I have performed is the Perspective transform. I have performed the perspective transform so as to have a "Birds eye view" of the lane lines so as detect the lane lines better with their curvature.
Below are the source and destination points that I have taken for the perspective transform. I have chosen the source and destination points manually and have arrived at the final result after tweeking them for the best result.

The resultant source and destination points are:
Here the width and height are the width and height of the image.

|Source        | Destination   | 
|:-------------:|:-------------:| 
| 590, 450      | 300, 0        | 
| 700, 450      | 850, 0      |
| 280, 710     | 280, 710|
| 1080, 710      | 850, 710|

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.
Below is the result of the warped images:

![alt text][image12]

I have also used the averaging technique average out the path from the previous 20 frames so the detection of the lane lines does not move out of the actual lane lines

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The next thing aim I had was to draw the lane line properly on the image. For that I plotted a histogram for the final resultant images obtained after combining the gradient and color thresholding.
Below is a visualization of a histogram that I obtained for one of the test images. 

![alt text][image6]

The I used the sliding window technique to find the lane lines and fit it to a second degree polynomial so as to obtain a smooth line for the lane.
Below is the result that I obtained after the sliding window technique. 

![alt text][image8]
 
#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I calculated the radius of curvature of the lane in the same way specified in the classroom. Below are the steps that I followed to calculate it.
```python
y_eval = np.max(ploty)
    font = cv2.FONT_HERSHEY_SIMPLEX
    ym_per_pix = 30/720 # meters per pixel in y dimension
    xm_per_pix = 3.7/700 # meters per pixel in x dimension

    # Fit new polynomials to x,y in world space
    left_fit_cr = np.polyfit(ploty*ym_per_pix, left_fitx*xm_per_pix, 2)
    right_fit_cr = np.polyfit(ploty*ym_per_pix, right_fitx*xm_per_pix, 2)
    # Calculate the new radii of curvature
    left_curverad = ((1 + (2*left_fit_cr[0]*y_eval*ym_per_pix + left_fit_cr[1])**2)**1.5) / np.absolute(2*left_fit_cr[0])
    right_curverad = ((1 + (2*right_fit_cr[0]*y_eval*ym_per_pix + right_fit_cr[1])**2)**1.5) / np.absolute(2*right_fit_cr[0])
```
#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

To plot back the result onto the original image I have unwarped the image by using minverse and then combined it back with the original image.
Below is the final result that shows the lane lines marked on the test images.
 
 Original Test Images:
 
![alt text][image7]

Lane Line Marked On the Images:

![alt text][image9]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_video/project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

While working on the project some of the problems that I faced were for finding the correct set of source and destination points to find a good perspective transform for the images.

At a place in the video the detection goes out a bit from the lane line as the lane lines are not visible properly at that place. to make it work better I can try to tweak the color thresholds and try directional gradient instead of the magnitude of gradient that ii have used in my pipeline..

There are some lines that are little wobbly that might be due to the thresholds that I have chosen for the color and gradient threshold. The pipeline may fail at the places where there is a transition from shadow to under very bright light or moving out of very bright light as the lane lines are not that properly visible from far. 

To make it more robust I can try to tweak the parameters and try changing the pipeline that I have used. I can also try to use other color spaces than HSL. 
