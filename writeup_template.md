## Self-Driving Car Nanodegree

### Write-up

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

[image1]: ./output_images/Draw_cardboard/draw15.jpg "ChessBoard calib"
[image2]: ./output_images/Undistort_images/undist7.jpg "Undistort"
[image3]: ./output_images/Thresholding/thresh7.jpg "Threshold"
[image4]: ./output_images/warped/topview7.jpg "WarpPerspective"
[image5]: ./output_images/lane_detect/lane7.jpg "Sliding window lane detector"
[image6]: ./output_images/pipeline/final7.jpg "Output"
[video1]: ./processed.mp4 "final"

## [Rubric]

writeup->done

camera calibration->done

Distortion correction->done

Thresholding->done

Bird's Perspective->done

Sliding window detector->done

Radius of curvature and offset calculation->done

project_video.mp4 processed->done

Discussions->done

---

### Writeup / README

In this section, I will be explaining the pipeline used in detail to achieve our goal of advanced lane detection.


#### Camera Calibration
I first defined object points and image points.I used a calibration chessboard to calibrate our camera.Using the chess board image found corners used it to update image points.Then this is used to get distance coefficient and camera matrix to model the perception of camera. This matrix is used to undistor images captured by the camera.

![alt text][image1]
![alt text][image2]

#### Thresholding

Next we use the undistorted image to extract certain features especially pertaining to lane lines.Here we will use a technique called thresholding.To be more precise we will be using a combination of sobel processing and channel thresholding.Sobel processing is very good for edge detection and channel thresholding is very good for selecting certain features of image associated with the channel.
The thresholding will use the following combination to acquire a clean lane lines.

1.sobel operation in X direction

2.sobel operation in y direction

3.sobel direction

4.sobel magnitude

5.channel thresholding in Red and green channel of RGB space

6.channel thresholding in Saturation channel of HLS space

A combination of or/and operation is used to combine results of 6 image processing described above.The end result is shown below.

![alt text][image3]

#### Bird's eye view

Here we will change the perspective of image.To make it simple , think of a rubber sheet this is the image.now you select some points on the sheet , you want to strech or compress the rubber sheet such that the selected points occupy the goal point that you decided these selected points to occupy.This is what is called perspective warping. Here we use the thresholded image and warp it in such a way you can see the road from topview.

![alt text][image4]

#### Finding lanes

Here we use an algorithm called sliding windows.When you see the image acquired in previous process,we observe bird's eye view of the road with parallel lanes.Now we need to find the lanes and classify them into either left or right lane.For this we use two small windows one for left and other for right .The start at bottom and slowly progress upwards.If the number of points detected in window is more than a threshold which is a hyerparameter , The sliding window reorients itself mean of the cluster.This trace of midpoints and points covered by sliding window give us information about the curve that will fit the lane.But there is a problem ! how do you know what should be the inital position of sliding window? We use histogram along the y axis of the image. The x axis position with high intensity would be the starting points of the left and right lane.

![alt text][image5]

#### Fit polynomial and calculate curvature.

Using the points classified into left or right lane in previous process we use polyfit function of numpy to fit a second order equation to the two clusters.Using the second order function given by polyfit we can trace the line overlapping the left and right lane.Now that we know the polynomial function of 2 lanes . we can calculate their respective curvature using 


            R_curve = ((1+(2Ay+B)^2)^(3/2))/|2A|

Where A and B are coefficients of the polynomial functions

Also all the values we calculate in above process in pixel space.To calculate curvature in real world space you can measure the width of lane in real life and observing the corresponding pixel length in image using both information we can find the conversion ration.In our case this resolution was used.For a road with lane width of 3.7m and lane length of 30m below is the conversion ratio.

            Xres = 3.7/800
            Yres = 30/720

using this ration will scale our polinomial line to realworld space and the function representing this line can be used to calculate curvature using above R_curve formula.

Now to calculate the mid position between the detected left lane and right lane at the bottom most edge of the image assuming thats where the car is with respect to image.we use below formula to calculate vehicle offset

    
            lanecenter_pos= (left_lane + right_lane )/2
    
            center_off= (vehiclepos - lanecenter_pos)* xres
            
Here center_off is vehicle offset.
        

#### 5. Visualisation

Finally now we have 2 polynomial functions each represent the curve fitting the left lane and right lane respectively.Now we create an empty image of same shape as our perspective warped image.Plot the lines and fill the space between the lines with shade.Now by using the information used in previous warp operation we sort of unwarp this image. This image is further combine with original image with help of opencv's weighted function.End result is the road and lanes correctly shaded with colors.We then use opencv's putText to display vehicle offset for each frame.

![alt text][image6]



---

### Pipeline (video)



Here's a [link to my video result](./processed.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I tested this pipeline with challenge video and it failed.The issue causing this error is due to presence of patches on the road.Even if threshold is altered,not sure if  sufficent readable information about lanes be extracted from image.Lets say if we really want to do this with thresholding then i believe HLS and YUV could help , but sobel must be avoided. But again with changing perspective of camera example difference in height of terrain can cause serious issue. I think its better to go with CNN based segmentation approaches than hand engineered ones as deep networks are pretty good in generalising features that represents a road.
