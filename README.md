## Writeup

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

[image1]: ./output_images/checker.png "Checker pattern"
[image2]: ./output_images/undistort.png "Undistorted Image"
[image2_]: ./output_images/undistort_nogrid.png "Undistorted Image wthout grid"

[image3]: ./output_images/birds_eye.png "Birds eye view"
[image4]: ./output_images/binary_thresh.png "Binary image"
[image5]: ./output_images/curve_fitting.png "Curve fitting"
[image6]: ./output_images/lane_marked_label.png "Final image"
[video1]: ./result.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the 3rd to 5th code cell of the IPython notebook located in "./advanced-lanelines.ipynb".  

I used opencv functions `findChessboardCorners` and `drawChessboardCorners` to identify the positions of the  corners of chessboard images taken from different angles with the same camera. The function `findChessboardCorners` returned me the locations of the corners in each image. `drawChessboardCorners` is used to draw the lines on the image to validate our results.

![alt text][image1]

Thus i build a corresponding set of 3d points and the 2d points which was used with the opencv function `calibrateCamera` to get the camera caliberation matrix and distortion coefficients. Then i used these values in opencv function `undistort` to undistort the images. An exmple is shown below. I have drawn the the grid lines to see how the image is getting transformed after undistortion.

![alt text][image2]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

A distortion corrected image is shown below.

![alt text][image2_]

#### 2. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform is shown in code cell 7. I have defined a function `find_perspective_matrix` which uses opencv's `getPerspectiveTransform` to return the perspective matrix when corresponding source and destination points are given . For finding corresponding points i downloaded an undistorted image of a straight road and used gimp software to mark the 4 points on the lane. I used this approach as i found that even small mistakes on the upper part of the image will get amplified in the output image. 

The source and destination points i used are:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 576, 462      | 200, 200      | 
| 708, 462      | 1080, 200     |
| 1068, 688     | 1080, 720     |
| 238, 688      | 200, 720      |

I verified that my perspective transform was working as expected by drawing the source points into the input image and observing show this line get transformed in the output image.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I experimented with different gradient thresholds and color channels. Initially my expeirment was with one operation only later i added multiple of this operation into my pipeline. The combination which worked best for me was, 
L Channel from LUV color space, with threshold between 225 and 255, followed by B channel from Lab color space, with threshold between 155 and 200. Refer code cells 9 and 10

![alt text][image4]



#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then to identify the lane lines i divided the image into two halves, one for the left lane and other for the right. On each half i isolated each row starting form the bottom. On each of this i identified the white pixels and found the mid of this white pixels. This was repeated for all the rows for both halves to get a series of points. An r2 polygon was fitted on these points to get both the lanes. Refer code cells 11 and 12.

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

As i have obtained the equation for the lane lines, i found the x intercept of both the curves at y = 720. Then averaged them. Now the distance from the center will be `image width/2 - average`. This value will be in pixels. Assuming that 3.7 meters is 700 pixels in image. I could find the distance in meters.

For calculating radius of curvature in meters i used the following code:
```
ym_per_pix = 30./720 # meters per pixel in y dimension
xm_per_pix = 3.7/700 # meteres per pixel in x dimension
left_fit_cr = np.polyfit(lefty*ym_per_pix, leftx*xm_per_pix, 2)
right_fit_cr = np.polyfit(righty*ym_per_pix, rightx*xm_per_pix, 2)
left_curverad = ((1 + (2*left_fit_cr[0]*np.max(lefty) + left_fit_cr[1])**2)**1.5) \
                             /np.absolute(2*left_fit_cr[0])
right_curverad = ((1 + (2*right_fit_cr[0]*np.max(lefty) + right_fit_cr[1])**2)**1.5) \
                                /np.absolute(2*right_fit_cr[0])
```

Refer code cells 13, 14.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented all the processes from undistorting the image, birds eye transform, binary tresholding, polygon fitting and back projecting the polygon to the actual image in a single function called `mark_lane_pipeline` (code cell 13). This function takes an image and outputs the lane marked image with lane curvature and distance from middle as output.

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

`process_vid` is my pipeline function which marks lane lines on an imput image.
Here's a [link to my video result](./result.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The approach presented in this project does a fairly good job in identifying the lane lines on the road. Unlike the first assignment we are not assuming the lane lines to be straight, instead we assume them to be curved. 

Though my approach is quite robust to the given video, it maight fail when the lane lines are not clear also with drastic changes in light intensity. I also doubt the accuracy on this method when the road is not flat.