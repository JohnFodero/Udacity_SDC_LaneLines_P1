# **Finding Lane Lines on the Road**

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./test_images_output/solidYellowCurve.jpg "solidYellowCurve.jpg"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

The pipeline steps:
0. Tune the parameters of the following image processing algorithms on a set of test images. I utilized the ["widgets" library](https://ipywidgets.readthedocs.io/en/latest/#) to create an environment that simplified the parameter tuning. Different images could easily be loaded to generate different sets of parameters that could then be compared for use in the generalized algorithm. See the dev_zone notebook for code that was used to tune these parameters.
1. Apply gaussian blur to the image to remove noise (too much blur will make it)
2. Apply canny edge detection to filter the edges in the image.
3. Apply a region of interest  
  Only look for lines in the 'horizon' of the field of view.
4. hough  
   "draw_lines" strategy:
    1. separate lines into left/right (bounded by slope).  
    For each left/right:
    1. average slopes
    2. pick a point closest to the top of the region of interest
    3. compute the y-intercept
    4. solve for the point at the bottom of the frame (y=540)
    5. plot the line

![alt text][image1]


### 2. Identify potential shortcomings with your current pipeline

1. The blur is intended to remove noise that will create unwanted edges that pass through the canny edge detection filter. Too much blur will make it difficult to detect edges. An intermediate value is needed to balance noise reduction while maintaining strong lane marker edges. These constraints require a balance that may neither remove noise nor maintain edges completely.
2. The edge detection might not work well in low-contrast settings. Like the blur, these parameters are tuned to a select set of images. Low light, poor weather, varying road colors all would negatively effect the contrast and therefore the performance of this filter.
3. The region of interest is narrow. It won't do a very good job detecting lane lines that curve far into the horizon. If the planning algorithms require a long set of lane lines, this mask would need to be changed to allow for curving horizons. Additionally, intersections might require a 'wider' horizon to see further into a sharp right or left turn.
4. Hough line detection is useful to detect groups of straight edges, however, as the edges become more complex, more complex analysis of the output of this filter is required to generate a useful set of lane lines.
   1. The strategy for the "draw_lines" function is fairly simple. Sharp corners, turn lanes, lane splits, and other cars in the roi would degrade the performance of this algorithm.



### 3. Suggest possible improvements to your pipeline

1/2. Blur and canny edge detection could be improved with automated parameter adjustment. For example, as noise increases, blur could be used to reduce the noise, while adjusting the upper and lower canny thresholds to compensate. Alternatively, "competing" sets of filters with different parameters could be judged and the ideal method given a driving condition would be chosen.
3. ROI choice could be paired with feedback from vehicle, pedestrian, and other road objects to modify the area in which the Hough line detection searches. If, say, a vehicle is following ahead, we could occlude the vehicle to maintain lane detection performance.
4. Hough/draw_lines strategy improvements.
   - A simple improvement could be to group detected lines, and find the average of these groups. Then, comparisons could be made to determine which is the most relevant. For example, we could judge the group of lines and determine if an opposing line is detected, if not, perhaps it is an artifact.
    - Alternatively, we could use a weighted average to bias the average toward the lines with longer lines.
    - Another solution would be to utilize past data to "smooth" the output. Jumps in lane lines can be predicted (eg. a sharp shift to the left 10cm is unlikely, but a disappearing line (as a turn lane appears) is more common), therefore, this information can be used to provide a smoother output to controls, mapping, and planning.


### 4. Summary

This algorithm is very simple and can provide very useful localization to the vehicle on primary roads. It likely would require a lot of tuning to work properly on subdivision roads, where lane lines do not exist, rather they are either inferred or are defined by the curb of the road. These situations provide edges that commonly have very low contrast, so this implementation might not be the best choice. Relying on lidar data to detect curbs or machine learning to process images might be a better approach in these cases. The challenge case was a good example of how my approach had many immediate shortcomings. For example, I sort the lanes between left and right by their position from the center of the screen. However as the road curves, that center point shifts and will exclude many lines from analysis due to the simplicity of this approach.
