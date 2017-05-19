## Advanced Lane Finding Project
### Steven De Gryze. May, 2018

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

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in code cell 4 of the jupyter notebook.  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  In code cell 7, I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![chessboard](./report_images/chessboard.png)

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I applied the distortion correction to one of the test images (code cell 8):
![Distorted and undistorted test image](./report_images/distorted-image.png)

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
                  
I used only color thresholds to generate a binary image (thresholding steps at lines # through # in `another_file.py`) using an HLS color transform. I selected the yellow hues of the left lane divider separately from the right hues of the lane divider. The yellow hues are filtered by selecting pixels with 18 < H-channel < 22, 150 < S-channel < 255 and 100 < L-channel < 200. The white hues are selected by filtering pixels with L > 220. This was done in code cell 9. Here's an example of my output for this step (see code cell 10). The yellow filtered pixels are shown in yellow and the white filtered pixels in white.

![Thresholded image with yellow and white detected pixels colored differently](./report_images/thresholded.png)

In addition, in code cell 11, I create a movie of the binary thresholded images with the white/yellow color distinction for the two lanes. Here's a [link to the video of the thresholded images](./binary_video_output.mp4)

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform is located in cell 12. The code for the points I selected are copied below.

```python
src = np.array([[220,  self.img_size[1] - 20],
                [1100, self.img_size[1] - 20],
                [740,  self.img_size[1] // 1.75 + 70],
                [548,  self.img_size[1] // 1.75 + 70]], np.float32)
dst = np.array([[390,  self.img_size[1] - 20],
                [925,  self.img_size[1] - 20],
                [925,  self.img_size[1] // 1.75 + 70],
                [390,  self.img_size[1] // 1.75 + 70]], np.float32)
```


                  
This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 220, 700      | 390, 700        | 
| 1100, 700      | 925, 700      |
| 740, 481     | 925, 481      |
| 548, 481      | 390, 481        |

The following figure shows these points on one of the test images:
![Points used to calculate perspective.](./report_images/perspective_points.png)
 
I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image (code cell 13).

![Original and perspective-corrected image](./report_images/warped.png)

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

For the first image in the video, I used a sliding window approach (function `get_lane_pixels_sliding` in cell 14) to find the lane pixels. For subsequent images, I searched an area in the proximity of the lane pixels of the previous frame (function `get_lane_pixels_window` in cell 14). If there were less than 1000 pixels detected during binary thresholding, I reverted back to the sliding window approach for that frame. The code for this logic is located in function `detect_lanes` of cell 14.

After the lane pixels were detected, I fitted a second order polynomial (example created using cell 15).

![Example input images for polynomial fit in next figure](./report_images/original-poly.png)
![Polynomial fit of lane pixels after perspective transform](./report_images/polynomial-fit.png)

One can see in the right image that the polynomial fit is not always good. This is because further away in the image, pixels become more sparse due to the perspective. The fewer pixels are going to have more leverage on the curvature of the fit in the distance. With more time, I would have filtered this wobbly curvature out a bit more smoothly.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

In perspective corrected space, the lanes are about 520 pixels wide (see calculation in cell 16). Therefore, there are 3.7/520 m/pixels in the x direction. Likewise, there are about 30/500 m/pixels in the y direction.

I calculated the offset by assuming the camera is placed at the center of your dashboard, so that the center of the image is the position of the camera (and hence that of the car).

The center of the lane is the middle of the two predicted lane lines at the y position closest to the car. The offset can then be calculated by taking the difference between the position of the camera and the position of the middle of the lanes.

The actual calculation of the curvature is done in the `detect_lanes` function of the `LaneFinder` class (cell 14). I found values of curvature around 500 - 900m and offsets of 0.3 - 0.5m.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step also in the `detect_lanes` function of the `LaneFinder` class. Here is an example of my result on a test image (example create in cell 17):

![Example images with the lane mapped in green.](./report_images/map_lanes.png)

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./smoothed_project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

* I focused on getting the binary thresholding as close as possible. It was a good idea to try and threshold the yellow and white pixels separately. I had good success working in the HLS color space. I first tried using gradient images, but found that there was a lot of scatter and volatility using gradient images.
* Having good thresholding yielded very good results on the lane detection. However, there was still jumpiness and scatter in the lanes. Therefore, I applied a smoothing pass where I did not update the parameters of the polynomial fit if the difference to the last parameters exceeded a parameter-specific threshold. I selected each of the 6 thresholds for the 6 parameters (3 for the left lane and 3 for the right lane) by looking at the derivative (i.e., `np.diff`) of each of the 6 parameters over time (see Figure below). It is clear that after taking the diff, one can use a simply threshold to eliminate the outlying frames and use the last frame's parameters to back-fill.

![Time series of parameter 1 of polynomial fit for the left lane across all frames in the test video](./report_images/parameter1.png)

* The curve in the distance still has some slight scatter/movement. I think that is okay, since it is furthest away from the car and there is no issue with the car not staying on the road. If I had a bit more time I would look into what parameter of the polynomial is responsible for the curvature and apply better smoothing to that parameter.
