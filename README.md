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

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The first step is camera calibration in order to remove the distortion effects that the camera may have. For this I utilized the code from the lectures and used the images of chessboard in calibration folder to find the distortion coefficient and camera matrix, using `cv2.calibrateCamera()` function.
 
I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][camera_cal/test_undist1.jpg]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Using the obtained camera matrix and distortion coefficients, I applied it on one of the images in test folder, i.e on test1.jpg. I got the output as follows:
![alt text][output_images/test1_undist.jpg]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

For detecting the lane lines, I used HSL colorspace and sobel gradient thresholding. Firt , using cv2.cvtColor(), I converted the image from RGB to HLS colorspace. Then, applied sobel operator in x direction on the l-channel of the hls image. In order to create a binary image with detected lanes, I thresholded the scaled sobel image using the threshold range provided in function parameter, and then performed bitwise AND operation with the binary image obtained from s-channel of hls image. Similarly, other images from test set have been processesd and saved in the output folder.

![alt text][output_images/test0_processed.jpg]
![alt text][output_images/test1_processed.jpg]
![alt text][output_images/test2_processed.jpg]
![alt text][output_images/test3_processed.jpg]
![alt text][output_images/test4_processed.jpg]
![alt text][output_images/test5_processed.jpg]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

I have first extracted the image dimensions as h and w.(Height and width respectively.) Since, the relative location of lane lines remains almost same with respect to the camera, I choose following source and destination points.


| Source        | Destination   | 
|:-------------:|:-------------:| 
| 550, 460      | 0, 0          | 
| 740, 460      | w, 0          |
| 1280, 720     | w, h          |
| 128, 720      | 0, h          |

Below, is the example of the perspective transform applied on each one of the test images.

![alt text][output_images/test0_warped.jpg]
![alt text][output_images/test1_warped.jpg]
![alt text][output_images/test2_warped.jpg]
![alt text][output_images/test3_warped.jpg]
![alt text][output_images/test4_warped.jpg]
![alt text][output_images/test5_warped.jpg]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

In order to decide what pixels are categorized as left and right lane pixels, I have used histogram as suggested in lecture videos as well. Thus, I took the histogram of lower half of image, and used sliding window technique to assign the pixels as left lane pixels and right lane pixels. Most of the approach and code has been refereed from the code provided in quizzes during lectures of Advanced Computer Vision.

Refer to the figure below for the lane candidates detection and sliding window visulaization.

![alt text][output_images/sliding_window.jpg]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

It is done inside the get_polynomial() function in cell 9. As explained in the theory, the idea is to fit a second order polynomial curve to the detected lane pixels in the image. For lane center, we simply take the average of l_fit_x_int and r_fir_x_int , which denote the curves for lanes. 

And, car center position is simply taken as half of the image width. But since the camera could not be mounted everytime on the center, the center offset value for the car can be calculated by using folowing formula: (car_center_pos - lane_center_pos) * x_unit_pix / 10.

However, we need to take into account the meters in actual represented in the image pixels(thus we need y_unit_pix and x_unit_pix), which denote the meters per pixel in y and x direction respectively.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

Once, the lane pixels and turns have been detected, next step is to use inverse perspective transform to superimpose the results back to the original image. And finally, use cv2.fillPoly() function for marking/coloring the detected lane in each frame of the video. 

![alt text][output_images/lane_detected_1.jpg]
![alt text][output_images/test3_final_lane.jpg]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

For outputs, I have saved the videos for each of three input videos provided. These output videos can be found in the folder test_videos_output. 

For example, here is the link to the output video of project_video.mp4 [link to my video result](./test_videos_output/project_video.mp4).Refer to Advanced_Lane_detection.ipynb for the processing pipeline for this video.
For the challenge video, output can be visualized here.[link to my video result](./test_videos_output/challenge_video.mp4)Refer to Advanced_Lane_detection-Challenge.ipynb for the processing pipeline for this video.

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I faced a lot of problem in detecting the lane lines for challenge video.There is difficulty specifically in the lanes under bridge and shadow of the trees. This is because the hue(underlying) color is hard to detect with colorspaces in such case, and gradient thresholding detects the wall boundries under bridge and the walls between the lanes on highway. If one particular value of threshold works for one frame, it doesn't perform well similarly on other frames of the video where such challenging scenario might appear.

Hence, for my lane detection pipeline, I was able to do perfect lane detection for project_video.mp4, somewhat good lane detection for challenge_video.mp4. For challenge video, I had to select other threshold ranges as compared to previous one, hence I have included the code for challenge video in other notebook file named Advance_Lane_detection-Challenge.ipynb. 

Finally, the approach I used for challenge video isn't working well for steep turns in the harder_challenge_video.mp4
