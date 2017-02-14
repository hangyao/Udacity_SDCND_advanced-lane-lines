# Advanced Lane Finding Project

## Udacity Self-Driving Car Nanodegree Project 4

---

The scope of this project is to develop a pipeline to process a video stream from a forward-facing camera mounted on the front of a car, and output an annotated video which identifies:

1. The position of the lane lines;

2. The location of vehicle relative to the center of the lane;

3. The radius of curvature of the road.

The pipeline created for this project processes images in the following steps:

1. Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.

2. Apply a distortion correction to raw images;

2. Apply a perspective transformation to warp the image to a "birds-eye view" perspective of the lane lines;

3. Apply color thresholds and gradient thresholds to create a thresholded binary image which isolates the pixels representing the lane lines;

4. Detect lane pixels and fit quadratic polynomials to find the lane boundaries;

5. Determine the curvature of the lane and vehicle position with respect to center;

6. Warp the detected lane boundaries back onto the original image;

7. Output visual display of the lane boundaries and numerical estimation of the lane curvature and vehicle position.

[//]: # (Image References)

[image1]: output_images/undistorted_output.png "Undistorted"
[image2]: output_images/undistorted_chess_output.png "Undistorted Chess"
[image3]: output_images/warped_output.png "Warped"
[image4]: output_images/warped_test_output.png "Warped Test"
[image5]: output_images/color_thred.png "Color Threshold"
[image6]: output_images/grad_thred.png "Gradient Threshold"
[image7]: output_images/combine_thred.png "Combined Threshold"
[image8]: output_images/fit_polunomial.png "Fit Polynomial"
[image9]: output_images/projected_lanes.png "Projected Lanes"
[video1]: output_images/project_video.gif "Video"
[video2]: output_images/challenge_video.gif "Video"
[video3]: output_images/harder_video.gif "Video"

---

## Install

This project requires **Python 3.5** with the following libraries/dependencies installed:

- [Numpy](http://www.numpy.org/)
- [Matplotlib](http://matplotlib.org/)
- [OpenCV](http://opencv.org/)
- [MoviePy](http://zulko.github.io/moviepy/)

You will also need to have software installed to run and execute a [Jupyter Notebook](http://jupyter.org/).

## Code

- `advanced-lane-lines.ipynb` - The main notebook of the project.
- `helper.py` - The script contained required helper functions.
- `line.py` - The script contained a required Python class.
- `README.md` - The writeup explained the image process pipeline and the project.

---

## Camera Calibration

The code for this step is contained in the first code cell of the Jupyter notebook located in "advanced-lane-lines.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I first applied this distortion correction to a chessboard image using the `cv2.undistort()` function and obtained this result:

![alt text][image2]

After visual verification, I applied the same distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

![alt text][image1]

## Pipeline (single images)

### 1. Distortion Correction

As mentioned above, I applied a distortion correction using the camera calibration matrix and distortion coefficients. A helper function `undistort()`, which appears in lines 8 through 19 in the file `helper.py`, is used to perform the distortion correction. The obtained distortion-corrected result on the test image is shown below:

![alt text][image1]

### 2. Perspective transformation

The code for my perspective transform includes a function called `warp()`, which appears in lines 21 through 47 in the file `helper.py`.  The `warp()` function takes as input an image (`img`). The source (`src`) and destination (`dst`) points are determined based on a straight-lines test image. I chose the following source and destination points:

| Source        | Destination   |
|:-------------:|:-------------:|
| 545, 460      | 0, 0          |
| 0, 700        | 0, 720        |
| 1280, 700     | 1280, 720     |
| 735, 460      | 1280, 0       |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a straight-lines image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image3]

Afterwards, a test image is warped using the same function.

![alt text][image4]

### 3. Thresholded Binary Image

I used a combination of color and gradient thresholds to generate a binary image.

I tried a variety of colorspaces and eventually used a combination of L-channel from HLS colorspace and b-channel from Lab colorspace. L-channel from HLS colorspace is very good to detect white lane lines, and b-channel from Lab colorspace is good to detect yellow lane lines. The color thresholding function `color_thred()` is at lines 49 through 72 in `helper.py`. Here's an example of my output for this step, which includes L-channel output from HLS colorspace, b-channel output from Lab colorspace, and the final combination of both channels.

![alt text][image5]

Regarding the Sobel gradient threshold, since the image was already perspective transformed, and I found that a Sobel gradient threshold on the x-direction is the most effective method to find the boundaries. The gradient thresholding function `grad_thred()` is at lines 74 through 89 in `helper.py`. Here's an example of my output for this step, even though it is not very good on this specific test image, but it was proven to be an effective addition once combined with the color thresholding.

![alt text][image6]

Then both thresholding methods are combined together, and the combine thresholding function `combine_threds()` is at lines 91 through 105 in `helper.py`. Here's an example of my output of combined thresholding.

![alt text][image7]

### 4. Identifying Lane-line Pixels and Fitting Their Positions with a Polynomial

This part of code includes 3 helper functions: `locate_lanes()`, `fit_ploy()`, and `fit_poly_plot()`, and they are at lines 107 through 300 in `helper.py`. Here I used the method of finding pixel intensity peaks in a histogram to locate lane lines. Then the sliding windows method was used to track lane pixels, and fitted with a quadratic (2nd order) polynomial.

The final output is like this:

![alt text][image8]

### 5. Calculating the Radius of Curvature of the Lane and the Position of the Vehicle with Respect to Center

I did this in 3 helper functions at lines 322 through 380 in my code in `helper.py`. `get_curv_m()` is used to calculate the radius of curvature in meters, and `dist2center_m()` is used to calculate the position of the vehicle with respect to center.

The assumptions of 30 meters vertical length and 3.7 meters lane width were applied to the pixel ratio, and radius of curvature and distance to center are therefore computed based on this scale. The final radius of curvature is averaged of the left and right lanes. To compute the distance to center, I assumed the camera is mounted at the center of the vehicle, and the optical axis of the camera is aligned with the centerline of the vehicle. Then the distance to center was calculated by comparing between the center of road from image and the center of image.

### 6. Plotting Back Down Onto the Road

I implemented this step in function `project_lines()` at lines 382 through 411 in my code in `helper.py`. The radius of curvature and the distance to center were also added to image as texts. Here is an example of my result on a test image:

![alt text][image9]

---

## Pipeline (video)

### 1. Final Video Output

To apply the pipeline to video processing, a class `Line()` was created in my code `line.py`. It also includes a method of this class `Line.update_line()` which is used to update the class fields once the lane lines have been validated.

Besides that, I wrote a function `validate_lane()` at lines 414 through 446 in `helper.py` to validate lane line using criteria of curvature, horizontal distance, and parallel.

And a pipeline function `process_image()` was created at lines 448 through 520 in `helper.py` integrating the whole pipeline process.

Here's a gif of the final video output.

![alt text][video1]

---

## Discussion

For generating thresholded binary image, I have only experimented limited number of combinations of color thresholding and gradient thresholding methods. For color thresholding, I've tried channels from RGB, LUV, HLS, and Lab colorspaces. For Sobel gradient thresholding, I've tried x-direction, y-direction, magnitude, and direction gradient. There are so many potential combinations, and other better thresholding methods may exist and may have more robust effectiveness.

For finding lane pixels, I only used peaks of histogram and sliding windows methods. These methods have been proven to be robust. However, there may be better methods which can improve robustness.

The lane line searching algorithm in this project is only good for searching both left and right lane lines. In some clips of video, when the vehicle is driving into a sharp turn, there is only one lane in the field of view of the camera. In such case the algorithm would fail to return valid results.

When the pipeline is applied to the challenge video, in some frames the algorithm fails to yield correct results. This means that the algorithm is not robust enough to handle the circumstances of driving in complex environments. It often fails at the vertical lines of pavement with strong contrast. I think it requires a better algorithm to threshold and to find lane pixels.

 ![alt text][video2]

When the pipeline is applied to the harder challenge video, it fails through the video. The poor lighting conditions are really tough to overcome, and the sharp turns also make the finding lane lines method failed.

 ![alt text][video3]

Another issue for this pipeline is that it may not be a general method. Many parameters such as those in perspective transformation and thresholding are determined under certain circumstances. They may not be able to be successfully used for other environments.
