# **Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./writeup_images/grayscale.jpg "Grayscale"
[image2]: ./writeup_images/edges.jpg "Canny Edge Detection"
[image3]: ./writeup_images/regionMask.jpg "Region Masking"
[image4]: ./writeup_images/houghRegionMask.jpg "Hough Transform"
[image5]: ./writeup_images/houghRegionMaskBestFit.jpg "Hough Transform Best Fit Lines"
[image6]: ./writeup_images/houghOverlay.jpg "Hough Transform Overlay"
[image7]: ./writeup_images/spuriousDetectionEffect.png "Spurious Detection Effect"
[image8]: ./writeup_images/histogramUnfiltered.jpg "Unfiltered Histogram"
[image9]: ./writeup_images/histogramFiltered.jpg "Filtered Histogram"
[image10]: ./writeup_images/postFiltering.png "Final Output"


---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of five steps: grayscale conversion, canny edge detection with median automatic thresholding, region masking, hough transform for line detection, and line drawing using least squares fitting to find the optimal line to draw.

The first stage of the pipeline is to transform the image from the RGB color space to one more convenient to work with.  Grayscale simplifies the representation of the image to relative intensities, which provides the majority of information we need to perform most signal processing tasks.  Below is an example of a grayscale image.

![alt text][image1]

Next, Canny edge detection is used to find the edges in the image.  The Canny algorithm finds the intensity gradient of the image.  It uses this to find strong edges, which are defined as values of the gradient which are above a threshold.  The threshold is calculated automatically using a ratio of 1:3 of the median value of the gradient.  Using a threshold ratio based on the median provided solid edge detection performance as shown below.

![alt text][image2]

After the edges have been found, region masking is applied to limit the image to a field of interest bounding the lane lines.  The region masking applied to the edges image is displayed below.

![alt text][image3]

With the edges detected and the image region masked, the next step will be to detect lines in the image.  The Hough Transform is the technique used to do this.  The Hough Transform converts lines from the image space (note: x, y coordinates of each pixel in the image) to the Hough Space (note: a points radial distance from the origin and it's angle off the x axis).  In the Hough Space, a point will be represented as a sinusoid representing all possible lines that could contain that point.  If sinusoids intersect in Hough Space, a line that contains the points described by the sinusoids is found.  The detected lines are shown below.

![alt text][image4]

Lastly, best fit lines representing the lane lines are drawn on the image.  The Hough Transform outputs the coordinates of a pair of points describe the lines detected.  The coordinates are taken to find the slope of each line and partition the lines into lines left of the center of the image and right of the center of the image.  Next, a linear least squares fit is performed to find the meta-lines, which best represent the trend of the left and right lines.  This seemed to provide decent performance, as shown below.

![alt text][image5]

Overlaying these lines on the original image, yields:

![alt text][image6]

However, this technique did not seem to perform so well on the yellow lane line video.  Spurious detections seemed to really throw the lines off, as shown below.

![alt text][image7]

Histograms for both the left and right slopes shows small extreme peaks caused by spurious detections.  

![alt text][image8]

Filter values were chosen between the peaks of the distributions to perform outlier rejection.  Applying these filter values yielded the following histograms.

![alt text][image9]

This filtering fixed the problem.

![alt text][image10]


### 2. Identify potential shortcomings with your current pipeline

My pipeline is prone to spurious detections.  I used a slope filtering technique to achieve solid performance on the videos for the assignment, but this technique would not generalize well.

From testing my pipeline on the challenge video, it clearly has problems with shadows, debris on the road, and curved lines.

### 3. Suggest possible improvements to your pipeline

The pipeline could be improved by more careful tuning of the hough transform parameters.  Although, after spending quite some time playing with them, I ultimately felt I achieved an acceptable level of performance.

Additionally, using a different color space like HSV could likely help avoid the problems with shadows and road debris.

Lastly, in order to handle the curves, the current pipeline seems like it would need an overhaul.  The region masking is fine for the videos given, but a sharp curve would be problematic.  Portions of the lane immediately in front of the vehicle would likely fall outside the region of interest, while picking up a lot of objects that you would not care about (ie. lanes on the other side of the road).  Estimating the radius of curvature and using that to update the region masking would likely help.
