# Self-Driving Car Engineer Nanodegree
## Project 1: **Finding Lane Lines on the Road**

<img src="examples/laneLines_thirdPass.jpg" width="480" alt="Combined Image" />

When we drive, we use our eyes to decide where to go.  The lines on the road that show us where the lanes are act as our constant reference for where to steer the vehicle.  Naturally, one of the first things we would like to do in developing a self-driving car is to automatically detect lane lines using an algorithm.

This project will detect lane lines in images using Python and OpenCV.  OpenCV means "Open-Source Computer Vision", which is a package that has many useful tools for analyzing images.  


The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report

### Image References (some expectations on the function of this software)
<img src="examples/grayscale.jpg" width="480" alt="Combined Image" />
<img src="examples/line-segments-example.jpg" width="480" alt="Combined Image" />
<video width="320" height="240" controls>
  <source src="examples/P1_example.mp4" type="video/mp4">
</video> 

---
### Reflection

#### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline operates on a numpy image as input. It consisted of 6 steps:

1. (EXTRACT) Extract the white and yellow attributes of an image (via HSV conversion)
2. (REMOVE) Apply a gaussian blur on the image (via grayscale conversion) to remove image 
	noise
3. (EDGE DETECTION) Execute a Canny operation to find the edges of line features in the image
4. (CROP ROI) Create a ROI mask by cropping the image to only the field of view of the vehicle and the road, which
   is [almost] a trapazoidial shape.
5. (FIND LINES) Calculate the hough lines transformation on the cropped image to extract the common slope(s) of the
   lane lines. 
6. (FIT LINE AND DRAW) Iterating though each hough line, calculated the average slope of the left and right lines, left line 
   have a positive slope, right line having a negative slope. Using the averaged slopes, draw a left 
   and right lane lines as solid lines over the original color image.

Using the above pipeline, we determined the lane lines in jpeg images stored in ./test_images and had the output stored in ./testimages_output as PNG files. I confirmed that lanes lines were detected and solid lines were annotated over the original images appropriately.

I then converted the pipeline into a function that takes a input of numpy-image. Thus reading a video files, we can apply the lane finding pipeline to every frame in the video and write out a new video with the line overlays. Using the videos in ./test_videos, we determine the lane lines in each video, and write them to output files stored in ./test_videos_output

Using the project template, I refactored the following code:
	"read image code" section was refactored into 2 functions:
	find_road_lines() that read a image file, converted the file to a image and called find_road_lines_by_image
	find_road_lines_by_image() that takes an image and runs it against the pipeline producing a image that has lane line
	annotations overlayed on the image.

I also refactored the helper functions:

	I added draw_solid_lines to convert the resulting hough piecewise lines into a continuous solid line for both the left and right lans lines. I used the average slope method to determine the slope of the left and right lane line. I also incorporated a low pass filter to prevent sharp changes in slope due to false positives/noise. I also added in error handling logic in the event the lines were horizontal (slope = NaN or INF). This provides a smoother estimation of lane lines considering vehicles can't "jump" position.

#### 2. Identify potential shortcomings with your current pipeline

1. One potential shortcoming would be what would happen when the road has lots of curves or changes in elevation. Current slope estimate considers a good amount of distance away from the vehicle (field of view all the way to the horizon) to determine slope of the left and right lane lines. I added a variable, upper_boundary_y that can be used to shorten the horzion considered and solve the problem, but eventually will not work on extremely curvy or hilly roads.

2. Another short coming is lighting. This pipeline will not work in the dark (no headlights). Since I use feature extraction by color, yellows and whites may not be detected either by use of headllights, oncoming cars, direct sunlight etc...

3. Another short coming is other white or yellow objects in the field of view of the vehicle. Those would be detected as false positives and cause the lane lines to be draw improperly. The low pass filter was suppose to help avoid this.

4. Another short coming is optical occulsion. If the white or yellow lines are blocked by an object, lines cannot be determined using this method.

#### 3. Suggest possible improvements to your pipeline

1. A possible improvement would be to use background subtraction to identify only road surface, the initially crop an image with that added to the pipeline.

2. Another potential improvement could be to determine a better way calcuate slope of the left and right lane lines that can follow the curvature of the road as it starts to bend.

3. Another potential improvement could be to determine a better way to reject lane lines that fall obviously outside and off the road, possibly see if a lane line fits in or out of the cropped polygon ROI.

4. Another potential improvement could be to color process the image before entering it into the pipeline, either by appying HDR techniques or thresholding to enhance the white and yellow colors.
