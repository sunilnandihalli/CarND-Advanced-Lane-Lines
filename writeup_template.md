## Advanced Lane Lines

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

[image1]: ./camera_cal/calibration1.jpg "distorted chessboard used for calibration"
[image2]: ./myoutput/undistorted_chessboard.jpg "Undistorted chessboard"
[image3]: ./test_images/test1.jpg "lane test image"
[image4]: ./myoutput/undistorted_lane_test.jpg "undistorted lane test image"
[image5]: ./myoutput/unwarped_undistorted_lane_test.jpg "unwarped undistorted lane test image"
[image6]: ./myoutput/binary_image.jpg "binary image showing potential lane pixels"
[image7]: ./myoutput/color_image_with_fitted_line.jpg "colored image showing the fitted lane lines"
[image8]: ./myoutput/lane_region_marked.jpg "final image showing the lane region marked"
[video1]: ./myoutput/project_video_with_lane_region.mp4 "video with the lane region marked"
[image9]: ./myoutput/unwarped_undistorted_straight_lane.jpg "unwarped undistorted straight lane image"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in IPython notebook located in "advanced_lanes.ipynb" between line numbers 19 and 38 of the first cell.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 
![alt text][image1]
![alt text][image2]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image3]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at lines 74 through 83 in `advanced_lane.ipynb`).  Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image4]


#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `get_perspective_trf()`, which appears in lines 40 through 45 in the first cell of `advanced_lane.ipynb` `example.py`.  The `get_perspective_trf()` function takes nothing as input. It internally assumes a source and destinaiton points.  I chose the hardcode the source and destination points in the following manner:

```python
    src = np.asarray([[295,660],[1023,660],[766,500],[518,500]],dtype=np.float32)
    dst = np.asarray([[295,660],[1023,660],[1023,500],[295,500]],dtype=np.float32)
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 295, 660      | 295, 660        | 
| 1023, 660      | 1023, 660      |
| 766, 500     | 1023, 500      |
| 518, 500      | 295, 500        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image9]
![alt text][image5]
#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I transformed the unwarped image to HLS space. Thresholded the s-channel to the range 90-255 which resulted in a binary image showing pixels which could potentially be the lane-line. I took the original image and transformed it into gray-scale and applied gradient in the x direction and scaled the gradients to the range 0-255. I thresholded the gradients with the range 20-100. This again resulted in a binary image. I combined the previous two binary images by ORing each pixel-location. This resulted in a combined binary image. The corresponding code is in the first cell of the advanced_lane.ipynb between lines 74-83

A histogram was drawn counting the number of pixels along each column. The peaks of the histogram was used to identify the base of the lane-lines (code 66-71 of advanced_lane.ipynb cell-1). 

Then I assumed a small-window around the base region of each lane-lines and collected all the pixels which were identified as potential lane pixels by the earlier combined-binary images and averaged to obtain the base of the lanes. This window was then progressively either kept at the same location if we didn't find enough pixels to reliably identify the lane-pixels or moved to average if there were enough pixels in the previous window. Once the windows were stacked up to the end of the image/horizon, we collected all the pixels identified by the windowing technique for each lane line. (code in lines 85-129 of first cell of advanced_lane.ipynb)

 Then a second order polynomial was fit through the pixels identified in the previous stage.

![alt text][image6]
![alt text][image7]
![alt text][image8]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines 131 through 134 in my code in `advanced_lane.ipynb`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines 187 through 207 in my code in `advanced_lane.ipynb` in the functions `draw_lane_lines()` and `draw_lane_region()`.  Here is an example of my result on a test image:

![alt text][image8]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./myoutput/project_video_with_lane_region.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?



Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
