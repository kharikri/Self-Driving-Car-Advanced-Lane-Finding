# Advanced Lane Finding Project
 
## Introduction

The goals of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images
* Apply a distortion correction to raw images
* Use color transforms, gradients, etc., to create a thresholded binary image
* Apply a perspective transform to rectify binary image ("birds-eye view")
* Detect lane pixels and fit to find the lane boundary
* Determine the curvature of the lane and vehicle position with respect to center
* Warp the detected lane boundaries back onto the original image
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position


## Rubric Points
---

### Files Submitted

#### 1. Submission includes all required files and can be used to obtain the final video

My project includes the following files:
* **P4_AdvancedLaneFinding.ipynb** contains the python code and in the following writeup all references to python code points to this file
* **writeup_report.md** summarizes the process used and results
* **output_images** directory containing output images shown in this writeup
* The final output video is on YouTube and linked at the end of this report

### Writeup/README

#### 1. Provide a Writeup/README that includes all the rubric points and how you addressed each one.

You're reading the Writeup.

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the 2nd code cell of the IPython notebook (**P4_AdvancedLaneFinding.ipynb**).

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result. The code for this is in the 3rd cell of the notebook: 

![alt text](https://github.com/kharikri/SelfDrivingCar-AdvancedLaneFinding/blob/master/output_images/test_dist_s.jpg)
![alt text](https://github.com/kharikri/SelfDrivingCar-AdvancedLaneFinding/blob/master/output_images/test_undist_s.jpg)

&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Original Distorted Image &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Undistorted Image

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.
I took one of the test images (`test1.jpg`) shown on the left and the undistorted image is shown on the right. To see the undistortion effect look at the dashboard of the car at left or the right bottom and you'll see that in the undistorted image it is slightly flattened. The code for this is in the 4th cell of the notebook.

![alt text](https://github.com/kharikri/SelfDrivingCar-AdvancedLaneFinding/blob/master/output_images/test1_dist_undist_s.jpg)

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
I used six different thresholds (X, Y, Mag, Direction, S, R). The first four of them are gradient and the last two are color. Then I output a binary image of these thresholds. The code for this is in the 5th and 7th cells of the notebook. The following picture shows all the six thresholds for `test1.jpg`:

![alt text](https://github.com/kharikri/SelfDrivingCar-AdvancedLaneFinding/blob/master/output_images/XMSThresh.jpg)
![alt text](https://github.com/kharikri/SelfDrivingCar-AdvancedLaneFinding/blob/master/output_images/YDRThresh.jpg)

As you will notice from the above picture, the X, S, and R thresholds filter the vertical lines better than the others. So I'll combine these three in the following to see which produces the sharpest binary.

![alt text](https://github.com/kharikri/SelfDrivingCar-AdvancedLaneFinding/blob/master/output_images/XSRThresh_s.jpg)

I used all these four combinations in the video pipeline and I got the best results for the X+S+R threshold even though the S+R combined binary looks the sharpest filtering out the trees and irrelevant details. This is because over the long run of the video under different conditions (shade, sun, trees, etc.) the X+S+R threshold performs better so I use that for my pipeline. The code for this is in the 8th cell of the notebook.

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp()`, which appears in the 9th cell of the IPython notebook.  The `warp()` function takes an image (`img`) as the input, undistorts it and uses the perspective matrix `M` and the function `cv2.warpPerspective()` to warp the image. The perspective Matrix `M` is obtained by choosing a trapezoid (`src`) which gets transformed into a rectangle (`dst`) with the function `M = cv2.getPerspectiveTransform(src, dst)`. I hardcoded the source and destination points. I applied the `warp`function to the test image and I got the following result. This picture shows the source (in blue) and destination (in red) points drawn out on the image. The lines in the warped image are parallel:

![alt text](https://github.com/kharikri/SelfDrivingCar-AdvancedLaneFinding/blob/master/output_images/warp_s.jpg)

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?
I have two functions to fit polynomials. The first function `initial_lane_polynomials()` is used for the very first frame of the video. The code is in the 10th cell of the notebook. Here I first find the histogram of the lower half of the image to find the pixel concentrations. If I did a good job of filtering with the thresholds I should see two peaks as shown in the following histogram:

![alt text](https://github.com/kharikri/SelfDrivingCar-AdvancedLaneFinding/blob/master/output_images/hist_s.jpg)

Once I identify the histogram peaks I roughly know where the lane lines are in the image. Using these peaks as the starting points I use the sliding window approach to fit the polynomials. In the sliding window method I collect the non-zero pixels along a line starting from the two histogram peaks all the way to the top of the image. After I collect the non-zero pixels along the height of the two histogram peaks, I fit a 2nd order polynomial (`Ay**2 + By + C`) through these pixels. I show the initial line fitting with the sliding windows in the left of the picture shown below.

The second line fitting function called `subsequent_lane_polynomials()` is used once I find a good initial fit. The code is in the 11th cell of the notebook. Here I do not go through the tedious process of finding the histogram and collecting the pixels through sliding windows. I simply start from the previous good fit and search around these polynomials within a margin. In the following picture on the right I show the subsequent polynomial fit. The green shaded area shows the margin where I search for the pixels.

![alt text](https://github.com/kharikri/SelfDrivingCar-AdvancedLaneFinding/blob/master/output_images/initial_line_s.jpg) &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;
![alt text](https://github.com/kharikri/SelfDrivingCar-AdvancedLaneFinding/blob/master/output_images/subs_line_s.jpg)

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The radius of the curvature of the lane is required to determine the suitable driving speed for the vehicle. The  [U.S. government specifications for highway curvature](http://onlinemanuals.txdot.gov/txdotmanuals/rdw/horizontal_alignment.htm#BGBHGEGC) gives the radius and speed specifications. For example, to drive at 45mph the minimum radius should be 246 meters.

To calculate the radius of the lane I first fit a circle of appropriate radius so that the curvature of the lane and the circle has the same tangent. Here is a visualization:

![alt text](https://github.com/kharikri/SelfDrivingCar-AdvancedLaneFinding/blob/master/output_images/radius.jpg)

Once I have such a circle than I simply calculate the radius of the circle which approximates to the radius of the lane. The math to calculate the radius is [here](http://www.intmath.com/applications-differentiation/8-radius-curvature.php). The code  (`radius_of_curvature()`) is in cell 13.

The lane offset is simply the difference between the midpoint of the lane and the midpoint of the car. Midpoint of the lane is obtained by the difference between the right and left lanes. Midpoint of the car is the middle of the image assuming the camera is mounted at the midpoint of the car. The code (`lane_offset()`) is in cell 14.

For both the radius and the lane offset I convert the dimensions from number of pixels to meters.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in cell 16. Here I warp back the polynomial lines using inverse perspective matrix (`Minv`) and combine them on the original image. Here is an example of my result on a test image. This image also shows the radii and lane offset in meters.

![alt text](https://github.com/kharikri/SelfDrivingCar-AdvancedLaneFinding/blob/master/output_images/fnl_img.jpg)

The above six steps are implemented in a pipeline in `process_image()` (implemented in cell 19). It takes one image at a time and does each of the above steps in sequence.

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

The video pipeline in cell 25 simply runs the image pipeline (`process_image()`) in sequence by feeding one image at a time. Here's the [link to my video result from first submission.](https://www.youtube.com/embed/TX92Vc7ZQl4?ecver=1). Here is the [video link for my second submission](https://www.youtube.com/embed/leiNx8SJiLs?ecver=1).

I've made the following changes to make the polynomials track the lane more accurately:
1. In my previous submission the threshold filter was not doing a good job when there was shade and sudden changes in light. In my current version, I use a combination of six thresholds (in cell 8) which does well in both normal light as well as under shade making my line detection more accurate
2. In the previous implementation I did not have any smoothing of coefficients. In this version, I smooth (average) coefficients (in cell 18) across five most recent good lines/polynomials. If I do not detect a good line I use the previous polynomial

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

My algorithm (in cell 18) for finding polynomials is as follows:
1. Use the sliding window approach to find the initial polynomial for the first frame
2. From the second frame onwards I use a targeted search to find subsequent polynomials
3. However if for five continuous video frames the polynomials do not satisfy the sanity check, I go back to step one and start looking for a new polynomial with the sliding window approach

The sanity check (in cell 12 of the notebook) simply checks to see if the lane lines are roughly parallel. This uses another function called `lane_width()` (in cell 15) to calculate the width of the lane.

This simple approach works for the assignment video -- `project_video`. However, this does not do a good job for the challenge video as this video has newly laid asphalt on the road and my algorithm confuses the edge of the newly laid asphalt as a lane. This needs to be  fixed at the histogram stage by accounting for reasonable width of the road.

My overall impression is lane detection with computer vision is fairly straightforward and predictable (unlike DNNs), but to make it work for every road condition will become exhausting. A combination of computer vision and DNN might be an approach to take.
