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

[image1]: ./output_images/undistorted_output.png "Road Undistorted"
[image3]: ./output_images/sobel_output.jpg "Binary Example"
[image4]: ./output_images/warp_output.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./output_images/lane_overlay_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the section 'Calibrate Camera' in the IPython notebook "Advanced Lane Lines.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function. The calibration coefficients are stored in a pickle file for reuse.

I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image1]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at in the section named 'Sobel').  Here's an example of my output for this step.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `calibrate_perspective_transform()`, which appears in the section 'Perspective/Warp Transform' which computes the coefficients and stores them in a pickle file for reuse. The `warp_perspective()` function takes as inputs an image and uses the coefficients to warp the image. The section also has another function named `unwarp_perspective()` for doing the reverse. I chose the hardcode the source and destination points in the following manner:

```python
  bottom_left = (189, 720)
  top_left = (591, 450)
  top_right = (689, 450)
  bottom_right = (1125, 720)
  
  gray = cv2.cvtColor(ud_image, cv2.COLOR_RGB2GRAY)
  src = np.float32([top_left, top_right, bottom_right, bottom_left])

  width = gray.shape[1]
  height = gray.shape[0]
  offset = 250
  dst = np.float32([[offset, 0],
                    [width-offset, 0],
                    [width-offset, height],
                    [offset, height]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 591, 450      | 250, 0        | 
| 689, 450      | 1030, 0       |
| 1125, 720     | 1030, 720     |
| 189, 720      | 250, 720      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The functions to detect the lane lines are in the section 'Finding Lane Lines'. There are two functions `find_lane_inds_from_scratch()` and `find_lane_inds_from_previous()` - the first one uses the windowing method to find lanes from scratch while the second one utlizes the left and right fit from the previous frame to identify the lane lines. The `find_lane_fit()` uses the two functions to find the left and right lane fit. I reused a lot of code implemented in the chapter 15 here.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The code to determine the curvature radius is also implemented in the section 'Finding Lane Lines' under the function `find_lane_curvature()`. 

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented utility functions in the section 'Finding Lane Lines' named `draw_lane_zone()` and `draw_additional_info()`. The `draw_lane_zone()` function draws a polygon using `cv2.fillPoly` to mark the detected lane area. And `draw_additional_info()` computes the curvature radius and the distance from center as a overlay on the image. Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_processed.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I wanted to mention a couple of things I had to tune - one was the range of road I used for the perspective transform. The first one I used lead to poor lane detection coupled with the optimization for lane detection using the previous fit. I had to adjust the range to get this to work properly.

Couple of ideas for improvement:
1. Add additional guard rail checks in the lane detection to ensure that the lane line are better especially when the color of the road changes (blacktop to cement)
2. Rank the two lane lines based on confidence and use the better lane to influence the other lane line (use the fact that the lines will be parallel to each other for the most part) and there by improving the lane detection when one of the lane line is weaker.
