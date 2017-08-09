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

[image1]: ./images/camera_calibration.png "Undistorted"
[pipeline0]: ./pipeline/original.jpg "Original"
[pipeline1]: ./pipeline/undistored.jpg "Undistored"
[pipeline2]: ./pipeline/binary.jpg "Binary"
[pipeline3]: ./pipeline/bird_eye_view.png "Bird eye view"
[pipeline4]: ./pipeline/polynomials.png "Polynomials"
[pipeline5]: ./pipeline/result.jpg "Result"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "Advanced Lane Finding.ipynb". 

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![pipeline0]

To correct the distorition I applied the `cv2.undistort()` and got the follwing result: ![pipeline1]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of gradients, direction of the gradient and the grayscale threshold to generate a binary image (see the `process_image()` function in the notebook).

```python
    binary_dir = dir_threshold(undist, sobel_kernel=5, thresh=(0.7, 1.3))
    binary_x_s = sobel(undist, orient='x', channel='s', sobel_kernel=9, thresh=(20, 80))
    binary_x_l = sobel(undist, orient='x', channel='l', sobel_kernel=9, thresh=(20, 80))
    
    binary_gray = gray_thresh(undist, thresh=(180, 255))
    
    # Combine the results
    binary = np.zeros_like(binary_x_s)
    binary[((binary_x_s == 1) & (binary_x_l == 1)) | ((binary_dir == 1) & (binary_gray == 1))] = 1
```

  Here's an example of my output for this step.

![pipeline2]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `camera_to_bird_eye_transform()`, which appears in the notebook file `Advanced Lane Finding.ipynb`.  The `camera_to_bird_eye_transform()` function takes as inputs an image (`img`).  I chose the hardcode the source and destination points in the following manner:

```python
    offset_x = 290
    offset_y = 0

    height, width = img.shape

    src = np.float32(
        [[585, 455],
        [698, 455],
        [188, height],
        [1123, height]])
    
    dst = np.float32(
        [[offset_x,offset_y],
        [width - offset_x, offset_y],
        [offset_x, height - offset_y],
        [width - offset_x, height - offset_y]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 455      | 290, 0        | 
| 698, 455      | 990, 0        |
| 188, 720      | 290, 720      |
| 1123, 720     | 990, 720       |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![pipeline3]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

To find the lane fit polynomials I created the `find_fit_polynomials()` function (see the `Advanced Lane Finding.ipynb`). This method uses the histogram to find the peaks, that are asossiated with the lanes. Then the peaks are used as a starting point to the sliding window that moves up to find the continuation of the lane. Having the data from the sliding window the 2nd order polynomial coefficients are found with the `np.polyfit()` function.

The lane approximation is drawn with the red color in the follwoing image:

![pipeline4]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I calculated the radius of curvature of the lane in the `calculate_curvature()` method in the notebook file.

I found the curvature for each line separately and then calculate the arithmetic average to find the lane curvature.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the `project_lane()` function in the notebook.  Here is an example of my result on a test image:

![pipeline5]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_processed.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The biggest challange for me in the project was to find the right combination of the methods (gradients, color thresholds, etc.) to create a reasonable binary image. There were many parameters, that seemed to work in one part of the video, but later there was a fragment of the road, with an obstruction or a shadow or a diffrent color, that caused noise in the binary image, what resulted in the incorrect lane finding.

To tackle that problem I collected a set of the problematic frames from the video, so I could experimet with diffrent hyperparamiters.

I was working mainly on the data from the main project video, so if there is a diffrent video, let's say recored in different weather conditions, my pipline is likely to fail.

To make the pipeline more robust I would need to collect more problematic frames (possibly from diffrent videos) and continue tuning the parameters and maybe adding some other methods, like gradient in a diffrent color space etc.
