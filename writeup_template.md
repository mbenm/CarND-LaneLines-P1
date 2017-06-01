# **Finding Lane Lines on the Road** 

### Reflection

### 1. Pipeline and resoning:

For this lane detection project, my pipeline consisted of 5 main steps, in order to allow extracting the lanes in both images and videos.
 To begin with, images are read and being preprocessed, in order to highlight the features of interest. In this case, these features are the lanes on the road.

One way to highlight these lanes is to convert the original image to grayscale. Converting images to grayscale before applying a Gaussian blur gave allowed satisfactory results for all of the mandatory test materials of this project, but resulted in dissatisfactory results when processing the challenge video, due to changing shadows and color intensity.
 Therefore, an alternative conversion to HSV space before applying the Gaussian blur and a color mask to retain yellow and white colors among a certain interval allowed to solve this problem.

An example of a color highlighting of both white and yellow masks is illustrated in Figure 1.
[Figure 1]: ./test_images/output/color_mask_pp.jpg "Color mask"

Once the input image is preprocessed to enhance the features of interest, a canny edge detection filter is applied, that output a binary image which consists of object edges within the range of low and high thresholds.
[Figure 2]: ./test_images/output/masked_edges.jpg "Masked edges"
Furthermore, a bitwise mask is applied in order to maintain only edges within the region of interest. Given the fixed position of the camera filming the road, lanes are most likely to appear in a triangular area of the lower half of the image. A mask is applied to maintain edges in this region and ignore the rest. 

The following step consists of applying a hough transform on the masked edge binary image, given a set of parameters (rho, theta, voting_threshold, min_line_length, max_gap) in order to optimize line detection. Except for the angle theta, the same set of parameters was used on both images and videos for lane detection.

 Once segments of interest are being detected, a modified draw_lines() function is used to make a decision and trace the left and the right lanes.
By calculating the slope of each segment of the data of the returned lines from the hough_lines function, several decisions can be made:
Negative slopes define left lane segments and positive slopes belong to right lane segments.
A slope that is close to zero could be considered as noise or a distortion and should be ignored from the average slope calculation. In this projects, all slopes with the range of [-0,3, 0,3] were rejected from the average slope calculation.
Points x1 that belong to a lane, but found to be out of a typical range are excluded from the average slope calculation.
Once all slopes are calculated and only useful points are maintained, average slopes m1 and m2 are calculated using numpy.polyfit, which apply a linear regression on all set of points stacked on two lines arrays, and return the corresponding slope and y-intercept of each array.

Based on these slopes and their corresponding y-intercept, x coordinates are calculated for each left and right lanes, given their y coordinate.

Additionally, in order to preserve temporal consistency between adjacent frames of a video, a weighted mean operation is applied by giving more importance to previously calculated coordinates. This operation allows avoiding flickering and temporal inconsistency throughout a video.
Finally, given the output lines of the right and left lane, the last cog of the pipeline allows drawing the lines on the original image and return the result of the lane detection (c.f. test_images/output and test_videos_output).

### 2. Limitations:

Although Hough transform method presents a solid method to detect lines in noisy images, this method is not robust enough for lane detection under various intensity characteristics.
Drawing lines to connect detected segments present satisfactory results in straight lanes but fail to predict lanes shapes in curved roads.

### 3. Improvements:

A possible improvement of the lane detection method could be possible by projecting the camera view to a bird view space, where linear and logistic regressions could be easier to apply on the detected lines and thus an optimized lane prediction robust to colors and intensity variations.
Another potential improvement could be achieved using a careful pre-processing which allows enhancing features of interest and reducing noise impact on the output.
