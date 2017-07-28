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

[undistorted]: ./writeup_images/chessboard_distort.png "Undistorted Chessboard"
[undistorted_sample]: ./writeup_images/sample_distort.png "Undistorted Sample"
[color_spaces]: ./writeup_images/color_spaces.png "Color Spaces"
[binary_sample]: ./writeup_images/binary_sample.png "Binary Sample"
[source_area]: ./writeup_images/source_area.png "Source Area"
[sample_unwarped]: ./writeup_images/sample_unwarped.png "Sample Unwarped"
[sliding_window_search]: ./writeup_images/sliding_window_search.png "Sliding Window Search"
[sliding_window_histogram]: ./writeup_images/sliding_window_histogram.png "Sliding Window Historgram"
[sample_unwarped]: ./writeup_images/sample_unwarped.png "Sample Unwarped"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the fourth and fifth code cell of the IPython notebook located in "./Advanced-Lane-Lines.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:
![Undistorted Chessboard][undistorted]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![Undistorted Sample][undistorted_sample]
Here I apply the same distortion correction like with the chessboard image before.

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I started by looking at all all color spaces and there individual channels.
![Color Spaces][color_spaces]

I've decided to use both HSL's L channel and LAB's B channel which was particularly helpful with the yellow lane markings. I did not use a gradient threshold as I was quite happy with the result already

Here's an example of my output for this step.

![Binary Sample][binary_sample]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `unwarp()`, which appears in cell 6 of my notebook.  The `unwarp()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points. I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32([  (255, 680),
                    (577, 460),
                    (704, 460),
                    (1052, 680)])
width = 480
dst = np.float32([(width, h),
                  (width, 0),
                  (w - width, 0),
                  (w - width, h)])
```

The source area looks like this:

![Source Area][source_area]

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![Sample Unwarped][sample_unwarped]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I perform a sliding window search where I identify the lines and fit their positions to a second order polynomial as per what was taught in class.
I calculate a histogram of the bottom half of the image which peaks make the base x-positions for the lines. Which are roughly at quater width from the left/right edge of the image. I chose 10 windows to identify lane pixels where each new window is centered on the midpoint of the previous window, following the lines from bottom to top. After this I fit the line pixels inside the windows to a second order polynomial.

![Sliding Window Search][sliding_window_search]
![Sliding Window Histogram][sliding_window_histogram]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I fit each lane line to a 2nd degree polynomial. With the coefficients and [this](http://mathworld.wolfram.com/RadiusofCurvature.html) I can calculate the curvature in "world coordinates".

With the assumption that the camera is center-mounted on the car I calculate the intersection of both lane lines with the bottom of the image. The difference between their center and the center of the image is the offset.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I used polyfill and polylines

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
