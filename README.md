# Advanced Lane Detection on Highway

### This project detects lane lines on a Highway using an Advanced Computer Vision Pipeline using only OpenCV. It also localizes the vehicles location within the lane.

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

[image1]: ./output_images/undistort.png "Undistorted"
[image2]: ./output_images/undistort2.png "Road Transformed"
[image3]: ./output_images/rgb_thresh.png "RGB Colorspace"
[image4]: ./output_images/hls_thresh.png "HLS Colorspace"
[image5]: ./output_images/color_thresh.png "Color Threshold after Merge"
[image6]: ./output_images/final_thresh.png "Threshold after Merge"
[image7]: ./output_images/warped.png "Perspective Transform"
[image8]: ./output_images/poly.png "Polynomial"
[image9]: ./output_images/formula.png "Formula"
[image10]: ./output_images/final.png "Final"
[video1]: ./project_video-output.mp4 "Video"

### Camera Calibration

The code for this step is contained in the first 4 code cells of the IPython notebook located in `./Advanced Lane Detection.ipynb`.  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objpoint` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Distortion Corrected image

To demonstrate this step, I have described how I applied the distortion correction to one of the test images like this one:

![alt text][image2]

#### 2. Color Threshold Mask

I leveraged 2 color spaces for this task - `RGB` and `HLS`. After testing various combinations of different color spaces, these 2 color spaces detected the lane lines best. Note that the line can either be yellow or white. Result of the RGB color space can be seen below. It is also clear that the Red Channel has the most prominant lane lines.

![alt text][image3]

Result of the HLS color space can be seen below. It is clear that the Saturation Channel has the most prominant lane lines.

![alt text][image4]

After testing which are the best channels to threshold from, I created a threshold mask and merged the R and S channels by taking the intersection of both spaces. The result can be seen below.

![alt text][image5]

#### 3. Gradient Threshold

For gradient thresholding, I used 4 different masks and merged them. The masks were

* Sobel X & Sobel y - Taking Sobel filter across both the directions detects the edges in the image.
* Magnitude - I also took into consideration the magnitude of the gradient and only took the strongest to be sure there is an edge.
* Direction - The direction of the gradient also matters in our particular case, as gradient along the vertical axis won't help us.

The result of the Gradient Threshold merged with Color Threshold can be seen below.

![alt text][image6]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `perspective_transform()`, which appears in the file `./Advanced Lane Detection.ipynb`.  The `warper()` function takes as inputs an image (`img`), and transforms the image with the `src` and `dst` points defined in the function. 

The source and destination points taken are:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 545, 460      | 0, 0        | 
| 735, 460      | 1280, 0      |
| 1280, 700     | 1280, 720      |
| 0, 700      | 0, 720        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image7]

#### 4. Sliding Window to detect Lane Lines

After receiving the thresholded and transformed image, we still have to detect the lane and obtain a polynomial for both the curves. For this, I used a sliding window technique which finds the max white pixels in each window and the window is sliding on the histogram of pixels. After we get exact points in each window, we fit a 2nd order polynomial to it using `np.polyfit`. After we have the polynomial, we restrict the search of lane pixels around the polynomial, hence skipping the sliding window technique and speeding up the process.   

![alt text][image8]

#### 5. Determine Curvate of lane and vehicle position with respect to center

The curvature of the lines are calculated using the following formulas. This is implemented in the function `get_curve`.

![alt text][image9]

We localize the vehicle between the lanes by finding the difference between the ceter of image and center of lane. It is defined in the function `get_vehicle_pos`.

#### 6. Final Result

The final result of the pipeline can be seen below

![alt text][image10]

---

### Pipeline (video)

#### 1. Final Video

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### Key Project Work Learnings and Future Work

* The pipeline is still slow and might not work for live lane detection.
* There are a lot of hyperparameters which need to be tuned and will change with the environment.
* We can get better results using a Deep Learning pipeline for this project as this pipeline does not scale well for different environments.
* This line also has limitations in low light and adverse weather conditions.