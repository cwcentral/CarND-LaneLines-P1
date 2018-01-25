# Self-Driving Car Engineer Nanodegree
## Project 1: **Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"
[image1]: ./examples/line-segments-example.jpg "Lane Line Overlay"
[image1]: ./examples/P1_example.jpg "Example of expect result of software"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline operates on a numpy image as input. It consisted of 6 steps:

1. Extract the white and yellow attributes of an image (via HSV conversion)
2. Apply a gaussian blur on the image (via grayscale conversion) to remove specular 
	noise from the image
3. Execute a Canny operation to find the edges of features in the image
4. Create a ROI mask by cropping th image to only the field of view of the vehicle and the road, which
   is [almost] a trapazoid shape.
5. Calculate the hough lines transformation on the cropped image to extract the common slope(s) of the
   lane lines. 
6. Iterating though each hough line, calculated the average slope of the left and right lines, left line 
   have a positive slope, right line having a negative slope. Using the averaged slopes, draw a left 
   and right lane lines as solid lines over the original color image.

Using the above pipeline, we determined the lane line in jpeg images stored in ./test_images and had the output stored in output_images as PNG files. I confirmed that lanes lines were detected and solid lines were annotated over the original images appropriately.

I then converted the pipeline into a function that takes a input of numpy-image. Thus reading a video files, we can apply the lane finding pipeline to every frame in the video and write out a new video with the line overlays.

Using the project template, I refactored the following code:
"read image code" was refactored into 2 functions:
	find_road_lines() that read a image file, converted the file to a image and called find_road_lines_by_image
	find_road_lines_by_image() that takes an image and runs it against the pipeline producing a image that has lane line annotations overlayed on the image.

I refactored the helper functions:
I added draw_solid_lines to convert the resulting hough piecewise lines into a continuous solid line for both the left and right lans lines. I used the average slope method to determine the slope of the left and right lane line. I also incorporated a low pass filter to prevent sharp changes in slope due to false positives/noise. I also added in error handling logic in the event the lines were horizontal (slope = NaN or INF). This provides a smoother estimation of lane lines considering vehicles can't "jump" position.

If you'd like to include images to show how the pipeline works, here is how to include an image: 

![alt text][image1]


### 2. Identify potential shortcomings with your current pipeline


One potential shortcoming would be what would happen when the road has lots of curves or changes in elevation. Current slope estimate considers a good amount of distance away from the vehicle (field of view all the way to the horizon) to determine slope of the left and right lane lines. I added a variable, upper_boundary_y that can be used to shorten the horzion considered and solve the problem, but eventually will not work on extremely curvy or hilly roads.

Another short coming is lighting. This pipeline will not work in the dark (no headlights). Since I use feature extraction by color, yellows and whites may not be detected either by use of headllights, oncoming cars, direct sunlight etc...

Another short coming is other white or yellow objects in the field of view of the vehicle. Those would be detected as false positives and cause the lane lines to be draw improperly. The low pass filter was suppose to help avoid this.

Another short coming is optical occulsion. If the white or yellow lines are blocked by an object, lines cannot be determined.

### 3. Suggest possible improvements to your pipeline

A possible improvement would be to use background subtraction to identify only road surface, the initially crop an image with that added to the pipeline.

Another potential improvement could be to determine a better way calcuate slope of the left and right lane lines that can follow the curvature of the road as it starts to bend.

Another potential improvement could be to determine a better way to reject lane lines that fall obviously outside and off the road, possibly see if a lane line fits in or out of the cropped polygon ROI.

Another potential improvement could be to color process the image before entering it into the pipeline, either by appying HDR techniques or thresholding to enhance the white and yellow colors.

