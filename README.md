# Advanced Lane Finding

---

The goals / steps of this project are the following:

* Step 1: Compute the camera calibration matrix and distortion coefficients given a set of chessboard images. Apply a distortion correction to raw images.
* Step 2: Use color transforms, gradients, etc., to create a thresholded binary image.
* Step 3: Apply a perspective transform to rectify binary image ("birds-eye view").
* Step 4: Detect lane pixels and fit to find the lane boundary. Determine the curvature of the lane and vehicle position with respect to center. Warp the detected lane boundaries back onto the original image. Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.
* Step 5: Produce overlayed output for the given input video.

[//]: # (Image References)

[distortion_removal_1]: ./output_images/distortion_removal_1.png
[distortion_removal_2]: ./output_images/distortion_removal_2.png
[perspective_sample]: ./output_images/perspective_sample.png
[perspective_1]: ./output_images/perspective_1.png
[perspective_2]: ./output_images/perspective_2.png
[perspective_3]: ./output_images/perspective_3.png
[perspective_4]: ./output_images/perspective_4.png
[perspective_5]: ./output_images/perspective_5.png
[perspective_6]: ./output_images/perspective_6.png
[binary_1]: ./output_images/binary_1.png
[binary_2]: ./output_images/binary_2.png
[binary_3]: ./output_images/binary_3.png
[polyfit_fill_1]: ./output_images/polyfit_fill_1.png
[polyfit_fill_2]: ./output_images/polyfit_fill_2.png
[final_output]: ./final_output.mp4

---
Welcome to Advanced Land Finding!

## Camera Calibration

### Step 1. Describe how I computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

In this step, I used a series of chess-board images to calibrate the camera. Each chess-board images has 9x6 corners inside. I map these corners with a norm grid to calculate the camera calibration matrix and distortion coefficients.

Chess-board images can be found in the **camera_cal** folder. I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.


## Pipeline (single images)

### Step 1: Compute the camera calibration matrix and distortion coefficients given a set of chessboard images. Apply a distortion correction to raw images.

 I applied this distortion correction to the test images in **test_images** using the `cv2.undistort()` function and obtained this result:

![alt text][distortion_removal_1]
![alt text][distortion_removal_2]

### Step 2: Apply a perspective transform to rectify binary image ("birds-eye view").

This step transforms the undistorted image to a "bird-eye view" of the road, which is a view from above. To do that, we need to map two straight lanes to two parallel lines under bird-eye view. This will facilitate polynomial fitting to the lane lines and measuring the curvature in the next step.

This part takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points. I chose to get the the source and destination points from one sample image with a small adjustment for `dst`

![alt text][perspective_sample]

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 200, 720      | 200, 720      | 
| 600, 450      | 200, 0        |
| 700, 450      | 1000, 0       |
| 1100, 720     | 1000, 720     |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][perspective_1]
![alt text][perspective_2]
![alt text][perspective_3]
![alt text][perspective_4]
![alt text][perspective_5]
![alt text][perspective_6]

### Step 3. Use color transforms and Sobel gradients to create a thresholded binary image.

Now we convert the warped image to different color spaces and create binary thresholded images which highlight only the lane lines and ignore everything else. The following color channels and thresholds did a good job of identifying the lane lines in the provided test images:

1. The S Channel from the HLS color space did a fairly good job of identifying both the white and yellow lane lines, but had a tendency to get distracted by shadows on the road.
2. The L Channel from the LUV color space did an almost perfect job of picking up the white lane lines, but completely ignored the yellow lines.
3. The B channel from the Lab color space did a better job than the S channel in identifying the yellow lines, but completely ignored the white lines.
4. Sobel filter applied directly on images does not help. Magnitude of gradient is good but very nosiy in some cases.

Combination of thresholded images does a great job of highlighting almost all of the white and yellow lane lines.

The S binary threshold was left out of the final combined binary image and was not used in detecting lane lines because it added extra noise to the binary image and interfered with detecting lane lines accurately. After trying Sobel x,y and magnitude and direction of Sobel gradient filters, I feel that color space thresholding is more useful, although there is a room to incorporate with the magnitude of Sobel gradient. So I use only L and B channel. My experiment output for this step is below.

Test Image 1
![alt text][binary_1]

Test Image 2
![alt text][binary_2]

Test Image 3
![alt text][binary_3]

A more robust approach is discussed here https://medium.com/towards-data-science/robust-lane-finding-using-advanced-computer-vision-techniques-mid-project-update-540387e95ed3.


### Step 4: Detect lane pixels and fit to find the lane boundary. 

The combined binary image is used to isolate lane line pixels. Lane lines are detected by identifying peaks in a histogram of the image and detecting nonzero pixels in close proximity to the peaks. I fit a polynomial to each of the lane lines.

#### Determine the curvature of the lane and vehicle position with respect to center.

I use the equation of radius of curvature can be found at http://mathworld.wolfram.com/RadiusofCurvature.html.

#### Warp the detected lane boundaries back onto the original image. Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

The space in between the identified lane lines is filled with green color to highlight the driveable area on the road. The position of the vehicle was measured by taking the average of the x intercepts of each line.

![alt text][polyfit_fill_1]
![alt text][polyfit_fill_2]

---

## Pipeline (video)

### Step 5. Performance Review

Here's a link to my video result in Youtube (or final_output.mp4 in this repo). 

[![Final Output](http://img.youtube.com/vi/5r7IEZgGCTw/maxresdefault.jpg)](
https://youtu.be/5r7IEZgGCTw)

The accuracy of Radius of Curvature calculation is judged by seeing whether the radius comes close to 1 km (~ 1000 m) at the turning points. This is because the data were prepared with a car driving at this specific map location, and the author has already measured the radius physically. In my case, I did. Especially when the car goes straight, the radius jumps back to about 10km, which makes sense.

---

### Discussion

#### Discuss problems in implementation of this project.  Where will it likely fail and how to make it more robust?

The most important point to make the whole project good is the lane detection, and lane memory.

Lane detection determine which points in the images stay on a lane. It subsequently affect the judgement whether they belong to left or right lanes and lane polynomial fitting later. My lane detection can be improved further in some cases if magnitude of Sobel gradient can be incorporated. 

In cases where lane is too blur to detect, lane memory comes in handy. For example, I base on the lane detection of the previous image to improve the current detection.
