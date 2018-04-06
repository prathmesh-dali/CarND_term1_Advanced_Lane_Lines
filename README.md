## Writeup Template

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

[image1]: ./output_images/undistorted_chess_board.png "Undistorted"
[image2]: ./output_images/undistorted_lane_line.png "Undistorted"
[image3]: ./output_images/luminance.png "Luminance thresholded binary"
[image4]: ./output_images/saturation.png "Saturation thresholded binary"
[image5]: ./output_images/sobel_x.png "Sobel X thresholded binary"
[image6]: ./output_images/sobel_dir.png "Sobel Direction thresholded binary"
[image7]: ./output_images/thresholded.png "Sobel Direction thresholded binary"
[image8]: ./output_images/warped.png "Warped Image"
[image9]: ./output_images/final_image.png "Final Image"
[video1]: ./project_video.mp4 "Video"

---

### Writeup / README


### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

After getting camera calibration matrix and distortion coefficients from chessboard image and successfully verification of the undistorted image of chessboard I have applied  `cv2.undistort()` on one of the test image and following are the results
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of hsl color threshold (s and l),  sobel x and direction threshold to generate a binary image.  Here's an example of my output for this step.  

| Filter | Sample |
| --- | --- |
| Luminance Thresholded Binary | ![alt text][image3] |
| Saturation Thresholded Binary | ![alt text][image4] |
| Sobel X Thresholded Binary | ![alt text][image5] |
| Sobel Direction Thresholded Binary | ![alt text][image6] |

While combining all the threshold I am taking only region of interest from all the binary images and then combining them to form one thresholded image. After combining all these thresholds the final thresholded image is below

![alt text][image7]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`. The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 200, 720      | 300, 720        | 
| 600, 447      | 300, 0      |
| 679, 447     | 900, 0      |
| 1100, 720      | 900, 720        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image. After applying perspective transform on thresholded image the output is

![alt text][image8]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I applied sliding window search on first frame and second frame onwards I am searching points in region nearby the line obtained by `cv2.polyfit()` in previous 10 frames by taking mean of coefficients to avoid jittery effect and failuer of algorithm at certain places where after thresholding we are not able to find lane lines. Once I obtain polynomial coefficint I am generating correspoing x co-ordinates for all y's where y is image height for both left lane and right lane. Then by using `cv2.fillpoly()` I am drawing corresponding quadrilateral on new image. After drawing quadrilateral on new image I am taking inverse perspective transform to restore back drawn polygon in original dimensions so that it can be drawn on original image. Then I am combining these two images to draw the same quadrilateral on undistorted image. Following is the undistoirted image with quadrilateral drawn on it.

![alt text][image9]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

To calculate radious of curvature of the lane I using same points generated in previous step for drawing quadrilateral. Since here we have line equations in pixel unit we need to map those to real world dimentions. For these mapping I am using multiplying factor of 

for pixel in y diection 30/720 meters per pixel 
for pixel in x diection 3.7/700 meters per pixel 

after this mapping I am using `cv2.polyfit()` to regenerate the modified polynomial coefficient and then I am using formula specified in lectures to calculate the radious of curvature. While displaying radious of curvature I am taking average of both lane radious of curvature.

To calculate center offcet I am calculating mid of lane by taking bottom point mid and then I am taking difference between image mid and lane center to calculate center offset

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

Here is the image with plotted lane area

![alt text][image9]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

While working on project video there are few brighter frames in the video where my pipeline was failing. TO overcome this I am taking average of last 10 line coefficient as starting point while search operation. 

Better thresholding can be done to make this code work on challenge video. 

To make it failproof if no lane lines are detected in region search method we can go back to sliding window search to get lane lines.
