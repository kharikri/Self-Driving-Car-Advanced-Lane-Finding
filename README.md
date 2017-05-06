# Self-Driving-Car-Advanced-Lane-Finding

This is the fourth  project in the first term of the Self Driving Car Nanodegree course offered by Udacity.

In this project road lanes are detected using image processing and computer vision techniques. In the [first project](https://github.com/kharikri/SelfDrivingCar-LaneDetection/blob/master/README.md) we fit straight lines to the lane lines. Here we detect the lanes and fit a second order polynomial. The process includes calibration of camera to account for distortion introduced by camera lenses, filtering the image to detect the lane lines with accuracy, warping the image to get a bird's eye view (top view) perspective of the lane, finding the lanes and fitting a polynomial and warping the polynomial back onto the original image. 

You can check out the details on my implementation [here.](https://github.com/kharikri/SelfDrivingCar-AdvancedLaneFinding/blob/master/writeup_report.md)

I implemented this project in Python, OpenCV, and FFmpeg. 
