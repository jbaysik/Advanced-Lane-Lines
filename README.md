## Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

The Project
---

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

## Writeup / README

### Camera Calibration

#### Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The camera calibration code can be found in the following file: [camera_calibration.ipynb](camera_calibration.ipynb)

The camera calibration was computed by using the checkerboard images provided in the 'camera_cal/' folder. By converting the images to grayscale and using the 'cv2.findChessboardCorners' function to find the corners in a 9x6 grid, points on the images were recorded to use as reference for calculating the calibration matrix. By taking the corner points found and providing it to the 'cv2.calibrateCamera' function, a calibration matrix is generated to undistort images taken by the respective camera, as shown in the following example:

![Camera Calibration Example](output_images/camera_cal_example.png)

### Pipeline (test images)

#### Provide an example of a distortion-corrected image.

The following example image was created using in [lane_pipeline.ipynb](lane_pipeline.ipynb) in cell 9:

![Undistort Example](output_images/undistort_example.png)

#### Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image. Provide an example of a binary image result.

The process for generating binary images of the lanes can be found cells 10-15 of [lane_pipeline.ipynb](lane_pipeline.ipynb) (sections headlined as 'Color Threshold' and 'Sobel Threshold'). The code goes through some of the testing process I did for looking at different image channels and thresholds.

The final binary images are creating first creating binary mask threshold of the R-channel of an RGB image. Then, the code takes the L-Channel of an LAB, and combines the XY-absolute value sobel threshold with a magnitude and directional sobel threshold. The two resulting gradients of the R-Channel and L-Channel are then 'or' combined together. The following images display the different results created from the images in the 'test_images/' folder:

![Binary Image Examples](output_images/binary_image_examples.png)

#### Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The process for applying a perspective transform can be found in cells 16-19 of [lane_pipeline.ipynb](lane_pipeline.ipynb) (in the section headlined as 'Persepctive Transform').

By undistorting and analyzing the example test images of straight line roads provided in the 'test_images/' folder, I was able to determine source points for where the perspective transform would focus in warp. Correlating destination points were calculated from an offset that would generate an appropriately warped image. These source points and destination points were then passed into the 'cv2.getPerspectiveTransform' function to generate matrices to both warp and unwarp images to and from a birds-eye-view. The following provides an example of the warp to a birds-eye persepctive:

![Perspective Transform Example](output_images/perspective_transform_example.png)

#### Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The process for identifying lane-line pixels can be found in cells 22-24 of [lane_pipeline.ipynb](lane_pipeline.ipynb) (in the section headlined as 'Plot & Analyze Lane Lines').

To find the lane-line pixels, the test images were first undistorted, warped through perspective transform, and then filtered through a threshold algorithms to create a binary image (as described in the previous questions). After the binary-warped image is generated, a histogram is used to plot out and track where the majority of the points exist in the image, designating the likely spots for the left and right lines. Using those points as a starting position, the image is analyzed starting from the bottom to the top in segmented windows to analyze potential curves by shifting and stacking each subsequent window. By taking the resulting windows for each lane, a line curve is generating passing through the midpoints of each window using the 'np.polyfit' function. From there, a smoother line is generated by polyfitting the line throughout the image space. The following shows the test images are generated when going through the lane-line finding process:

![Analyzed Lines Examples](output_images/analyzed_lines_examples.png)

#### Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The process for caluclating the radius of curvature and the vehicle position in the lane can be found in cells 25-30 of [lane_pipeline.ipynb](lane_pipeline.ipynb) (in the section headlined as 'Calculate Curvature').

To calculate the curvature of each line of the lane, the line that was found from the previous section is mapped to real word space using the 'np.polyfit' function and converting the pixel values to real world measurements (30 meters per 720 pixels in the y-direction, and 3.7 meters per 700 pixels in the x-direction). From there, those fitted curves were used to calculate the curvature of the respected lines with the following forumla:

```
((1 + (2*line_fit[0]*y_max*ym_per_pix + line_fit[1])**2)**1.5) / np.absolute(2*line_fit[0])
```

where 'line_fit' is the generated fitted line in real world space, y_max is the maximum y-value of the linespace, which is converted to real world space with ym_per_pix (30/720).

The car position was caluclated by taking the center of the image as the position of the car. This was accomplished by taking the width of the image space and converting it to world space. From there, the center of the each line was calculated with the following formula:

```
line_fit[0]*y_max**2 + line_fit[1]*y_max + line_fit[2]
```

where line_fit is the genarted fitted line in real world space, and y_max is the maximum y-value of the linespace. By using the midpoints of each line, the center of the lane is calculated by averaging the two points. Once having the calculated values of the car in the image and the center of the lane in the image, the difference of those values provides the distance of the car from the center of the lane.

#### Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The following images provides results of the found lanes and their caluclated curvatures, along with car position with respect to the center of the lane:

![Pipeline Data Image Examples](output_images/pipeline_data_image_examples.png)

### Pipeline (video)

#### Provide a link to your final video output. Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!)

The process for generating the video and taking it through the lane finding pipeline can be found in [P2-Advanced_Lane_Lines.ipynb](P2-Advanced_Lane_Lines.ipynb).

The above mentioned file generated a video named 'output_video.mp4' and can be found packaged with the submitted files.

### Discussion

#### Briefly discuss any problems / issues you faced in your implementation of this project. Where will your pipeline likely fail? What could you do to make it more robust?

The majority of my issues was properly finding the proper gradients and thresholds that worked in a variety of lighting and shadows, as well as both darker and lighter roads. While the algorithm and combination of thresholds I put together I feel does a acceptable job, there are still times where there may still be difficulty accurately finding lane lines, especially on roads with more faded and/or sparce lane lines. One way to work around the failure of faded lines in hard to view scenarios would be increase the camera view to account for the full road to potentially track the curvature of the road in cases where the car's individual lane is hard to find. This could create a backup and provide a slightly more robust way of accounting for more road environments.