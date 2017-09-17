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

[image1]: ./examples/before_undistort.png "Distorted"
[image2]: ./examples/after_undistort.png "Undistorted"
[image3]: ./examples/test5.png "Test"
[image4]: ./examples/test5_correct.png "test corrected"
[image5]: ./examples/road-undistorted.png 
[image6]: ./examples/road-perspective.png  "Road perspective"
[image7]: ./examples/road-polyfit.png "Fitted curves"
[image8]: ./examples/road-output.png "Output"
[image9]: ./road-process.png "Complete process"
[image10]: ./road-annotation.png "Curvature annotation"
[video1]: ./result.mp4 "Video"


## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

Camera matrix and coeficients were calculated using cv2.calibrateCamera(), I used findChessboardCorners and drawChessboardCorners instead of doing it manually like in the course work. Once I got the mtx and dist values I just hardcoded them on my ipython notebook.

```
MTX = np.array([[  1.15662906e+03,   0.00000000e+00,   6.69041437e+02],
 [  0.00000000e+00,   1.15169194e+03,   3.88137240e+02],
  [  0.00000000e+00,   0.00000000e+00,   1.00000000e+00]])
DIST = np.array([[-0.2315715,  -0.12000537, -0.00118338,  0.00023305,  0.15641571]])
```

I then used the output objpoints and imgpoints to compute the camera calibration and distortion coefficients using the cv2.calibrateCamera() function. I applied this distortion correction to the test image using the cv2.undistort() function and obtained this result:

distorted picture:

![alt text][image1]

undistorted picture:

![alt text][image2]


### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

I used the cv2.undistort() to calculate camera calibration matrix and distortion coefficients. It can remove distortion of image and output the undistorted image.:

original picture:
![alt text][image3]

distortion-corrected picture:
![alt text][image4]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at lines # through # in `another_file.py`).  Here's an example of my output for this step.  (note: this is not actually from one of the test images)

Near cell #4 in a section called Thresholding the code using vertical and horizontal gradients as well as color transformations can be found. No magnitude or angle gradient was used.:

```
    gradx = abs_sobel_thresh(img, orient='x', sobel_kernel=ksize, thresh=(20, 190))
    grady = abs_sobel_thresh(img, orient='y', sobel_kernel=ksize, thresh=(30, 190))
    s_binary = filter_s_select(img, thresh=(150, 255))
    l_binary = filter_l_select(img, thresh=(190, 250))
    
    combined = np.zeros_like(s_binary)
    combined[((gradx == 1) & (grady == 1)) | (s_binary == 1) | (l_binary == 1)] = 1
    combined_gray = np.uint8(255*combined)
```

![alt text][image5]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The warping code can be found near cell 5, in a section called Perspective transformation:
```
    # Warp the blank back to original image space using inverse perspective matrix (Minv)
    newwarp = cv2.warpPerspective(lane, minv, (img.shape[1], img.shape[0])) 
    # Combine the result with the original image
    result = cv2.addWeighted(img, 1, newwarp, 0.4, 0)
```

![alt text][image6]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The XY polinomial fit was taken almost verbatim from the udacity course work. It uses a sliding window over the image to separate out good and bad left and right lanes.

Tne np.polyfit was used to obtain a second order polynomial.

The section near cell #6 is called Extrapolate lines and draw them on the screen 

![alt text][image7]


#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The way to calculate the curvature radius and the vehicle position is to calculate a new x / y curve based on meters instead of pixels and with that get a radius using a formula based on the derivative of the curve. With the lane positions it is possible to average them and use that to calculate the position of the road based on the road width (average equaling zero means the car is centered).

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

Here is an example of my result on a test image:

![alt text][image8]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

It seems that trees can sometimes causes problems and thus not that robust. Maybe more tuning on parameters would help on that. 

Also, noticed that using AWS with GPU is much faster than local CPU running. 
