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

[image1]: ./writeup_images/chessboard_Undistorted.png "Undistorted Chessboard"
[image2]: ./writeup_images/undistorted_test_image.png "Undistorted Test Image"
[image3]: ./writeup_images/threshold_binary_testimage.png "Binary Image"
[image4]: ./writeup_images/undistorted_warped.png "Warped Image"
[image5]: ./writeup_images/identify_lane_pixels.png "Straight Lane Polynomial Fit"
[image6]: ./writeup_images/identify_lane_pixels_curved.png "Curved Polynomial Fit"
[image7]: ./writeup_images/camera_lane_offset.png "Lane-Vehicle Offset"
[image8]: ./writeup_images/inv_warp_fill.png "Inverse Warped Lane"
[image9]: ./writeup_images/final_lane_hgihlighted.png "Final Output"
[video1]: ./output_video/project_video.mp4 "Project Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

This writeup is inteded to fulfil the above requirement. The code is implemented in the notebook __P2.ipynb__ , and is organized in the same sequence as this document and the rubric. The notebook will be referenced for explanations used in this document. 


### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

For the calibration step, I started with the code already provided. Made some minor changes to the code for displaying images.

The code prepares the "object points", which will be the (x, y, z) coordinates of the chessboard corners in the image. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I wrote a function `image_undistorted` which uses undistorts the image passed as argument, using `objpoints` and `imgpoints`, also provided as arguments. Next, using the `cv2.calibrateCamera()` function I obtained the parameters `mtx` and `dist` used next. I applied this distortion correction to a chessboard image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]


### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.


I verified in the last step that the undistortion works as desired. Now, I applied the undistortion to test_images provided.
Iterating through the folder I read each image using `mpimg.imread()` and call the function `image_undistort` defined in the last step.

The output images are written in the folder 'test_images_output'. One of them is displayed as an example.

![alt text][image2]


#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at lines # through # in `another_file.py`).  Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]



Code in section 2.2 perfoms the requirements of this step. A function `apply_threshold()` receives the image and applies the thresholds for HKS and Sobel. 
Sobel derivate is determined using function `cv2.Sobel()` on the lightness channel. Absolute values of the gradient are used.
Color threshold is applied on the saturation channel

The scaled Sobel `sx_thresh` and saturation threshold `s_thresh` values used are defined below

Sobel Threshold for Gradient: `(20, 100)`
Saturation Threshold for color: `(170, 255)`

Both the thresholds are combined into one binary image and returned.

![alt text][image3]



#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The implementation for this section is via a function `warp_image()` which accepts the image and `src` and `dst` points as arguments. The prespective transform matrix is obtained using the function `cv2.getPerspectiveTransform()`. For later use, M inverse is also determined at this point by switching  `src` and `dst` inputs. The `warp_image()` returns the warped image as output.

A helper function `draw_lines` is used to draw the lines connecting the source and destination points, over the image using `cv2.ine()`, and `cv2.addWeighted()`.


Following are the sources points used

```python
p1 = [588, 455]
p2 = [696, 455]
p3 = [255, 680]
p4 = [1065, 680]
src = np.float32([p1, p2, p3, p4])
```

Destination points are defined using a margin:

```python
dst = np.float32([[margin,0],
                  [image.shape[1]-margin, 0],
                  [margin, image.shape[0]],
                  [image.shape[1]- margin, image.shape[0]]])
```
The source and destination points were plotted over the image for verification.

![alt text][image4]



#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]


Using the binary warped image from the last step, I identify the lane pixels. A function `find_lane_pixels()` perfoms this step, using the histogram obtained by summation of binary pixel values. This is obtained in both the left and right half of the image for left and right lane respectively. The hyperparameters are defined to set a margin or 100 pixels for the summation and a threshold of minimum 50 pixels to be counted.

An array of the identified points is returned for left and right lanes. An image with these points plotted is also returned.



A different function `fit_polynomial()` calculates the curve fitting for the lane lines. It calls the `find_lane_pixels()` function and to receive the arrays for points. Then it uses `np.polyfit()` is used to fit a 2nd degree polynomial using the points obtained in the last step. It returns the polynomial coefficients for left and right lane curves.

![alt text][image5]

![alt text][image6]


#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The radius of curvature is calculated inside the function `measure_curvature()`. Inside the function we define the y point where we want to determine the curvature and use the polynomial coefficients to determine the left and right radii of curvature.

`calculate_offset()` function calculates the offset by taking the difference between mid-point of two lanes, and the mid point of the image.
The underlying assumption here is that the center of the image is aligned with the vehicle center.

![alt text][image7]


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.


The final step of this pipline was to inverse warp the lane markings back onto the original image. I used the `Minv` perspective transfrom matrix determined earlier. I defined function `warp_inv_lanes()` which takes both the images, the polynomial fit, the Minv, and the offset parameters to apply the inverse warping. 

Left and right lane points are stacked using `np.hstack()` and `cv2.fillPoly` is used to fill the area with `[0,255,0]` (green color).
`cv2.warpPerspective` is again used for the warping.

![alt text][image8]




The section 2.7 sets the function that acts a the pipline and calls all the necessary functions defined above, to process the image from start to end.
`set_param_values()` defines the necessary parameters, and then`pipeline()` calls the functions in correct order get the final image.

![alt text][image9]


---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Using the pipeline used for images, I set up a pipeline for videos. The function `pipeline` is passed as argument to the `clip1.fl_image()` where `clip1` is the video input.
The output of the video looks satisfactory

Here's a [link to my video result](video1)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

This project was more involved, compared to project 1. I was able to implement individual steps of the problem without much difficulty, but combining all of them into a single unit was harder problem. Had to sort some issues with argument handling, default inputs, and pre-defined parameters. This all had to be set up such that when the pipeline is called for a video, all the parameters are already set up.

As an example, the radius of curvature for the lanes was needed in meters for printing, but in pixels for plotting on the image. I handled this by defining the default `xm_per_pix` and `ym_per_pix` for the function call as 1, so that it returns in pixel by defualt and in meters when those arguments are provided during the function call.


The current pipeline works well for the provided video. However it runs into issues when working for the challenge video. The shadows override the lane markings due to higher change in contrast. This probably could be prevented by using temporal averaging, and use hue values from HLS for another threshold.



