#**Finding Lane Lines on the Road** 

##Introduction
The following document describes the steps required to identify lanes lines on an image or video of a road. The steps are based on self driving nanodegree program conducted by Udacity.

The document first describes the goal of the project, the process that was taken to identify the lanes along with the issues faced and how they were finally resolved. 

----

##Project Goals
The goals  of this project are the following:
* Make a pipeline that finds lane lines on the road : The project involved detecting lanes lines on 5 images and 2 videos. An extra challenge video was also provided along with the project.
* Reflect on the work in a written report

---

##Reflection

###1. Solution
Once the image is read it is passed to a pipeline to detect lane lines consisting pf the following series of steps :

* **Step 1 : Convert to Gray Scale**
The image provided is usually a colored image. Converting it into grayscale using openCV's function *cvtColor(...)* will convert it into a grey 8-bit grey image. 

```
#use RGB2GRAY is image is read with mpimg.imread()
cv2.cvtColor(img, cv2.COLOR_RGB2GRAY)
#Or use BGR2GRAY if you read an image with cv2.imread()
cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
```
The image will look like the following : 
[@Grey Scale  | center | img01]: ./examples/gray_scale.png

* **Step 2 : Apply Gaussian Blur**
Going forward we need to find the edges of the lanes within the image. As edge detection is susceptible to noise in the image, we would need to remove the noise by applying Gaussian filter.  To do so, we use *GaussianBlur(...)* function from openCV. Pass the 8-bit output from the previous step to the function.
```
cv2.GaussianBlur(img, (kernel_size, kernel_size), 0)
```
The image will look like the following:     
 [@ Image with Gaussian Blur  | center | img02]: ./examples/gray_scale.png

* **Step 3 : Detect the lane edges**
The greyed out and filtered image from the previous two steps is given to openCV's *Canny(...)* function. *Canny(...)* takes the 8-bit image along with low and high threshold for hysteresis procedure resulting in an image with edges detected.

```
cv2.Canny(img, low_threshold, high_threshold)
```
The image now looks like the following:
[@ Edges detected  | center | img03]: ./examples/edge_detection.png

* **Step 4 : Apply region mask**
The next important step is to select the region of interest. This is done be creating a region which encompasses the lanes. We then mask out the image from previous step resulting in the selection of region consisting of left lane, right lane and area in between. 
To begin with we need to define the vertices of the image.  Take a ration to select the upper left and right points.
 
```
 #Golden ratio for a rectangle is 8/ 5 (width/ height)
 ratio = 1 / (8/ 5)
 regions = [
            (100, ysize),
            ((1 - ratio) * xsize, ratio * ysize),
            (ratio * xsize, ratio * ysize),
            (xsize, ysize)
           ]
```
The image will now look like the following:
[@ Region masked  | center | img04]: ./examples/region_masked.png

* **Step 5 : Detect lines using Hough Transform**
Now the we have detected the edges and have the region of interest, we will go ahead and detect the lane line.  We use probabilistic Hough Line Transform which is a efficient implementation of the Hough Line Transform. The output is the extremes of the detected lines (x{0}, y{0}, x{1}, y{1}).
Line variables
For Hough Transforms, we will express lines in the Polar system. And the equation of line can be written as:
										
```
			r = x * cosθ + y * sinθ 
			where r and θ are polar co-ordinates.
```
To apply probabilistic Hough line transform we use openCV's *HoughLinesP(...)* function.

```
cv2.HoughLinesP(img, rho, theta, threshold, np.array([]), minLineLength=min_line_len, maxLineGap=max_line_gap)
```

* **Step 6 : Draw Hough Lines**
The lines which we got after applying probabilistic hough line transform will now be drawn. Before drawing the lines detect the slope of the lines to differentiate between left lane and right lane.

```
slopes = np.apply_along_axis(lambda line: (line[3] - line[1]) / (line[2] - line[0]), 2, lines)
        
```

The resulting image will look like the following:
[@ Hough Lines  | center | img05]: ./examples/hough_line.png

Now the next step is to extrapolate these lines to create a single line . This can be done by separating the x and y points for the line and using *np.linalg.lstsq* to retrieve the slope and intercepts. 

The slope and intercept are then used to find out the new values for y.

```
#get all the x points for the line
x = np.reshape(lines[:, [0, 2]], (1, len(lines) * 2))[0]
#get all the y points for the line
y = np.reshape(lines[:, [1, 3]], (1, len(lines) * 2))[0]
    
#The equation for a line is y = mx + c
#we can rewrite the equation as y = Ap, where A = [[x 1]] and p = [[m], [c]]
#and then use lstsq to get the value of p
A = np.vstack([x, np.ones(len(x))]).T
m, c = np.linalg.lstsq(A, y)[0]

x = np.array(x)
#using the the value of m and c get all the values of y
y = np.array(x * m + c)
```
We need to go ahead and collect the slopes and intercepts across all the lines and average it across the current and previous results. 

**NOTE:** In case of videos, average should be taken out across the frames of the videos. This will result in a stable line over the lanes.

Once the average slope, intercepts, minimum and maximum y have been derived, get the top and bottom left and right points.

```
#Using the minimum value of y get the top right and top left points for
#the lines , x = (y - c)/ m
top_right_point = np.array([((avg_min_y - avg_right_c) / avg_right_m), avg_min_y], dtype=np.int32)
top_left_point = np.array([((avg_min_y - avg_left_c) / avg_left_m), avg_min_y], dtype=np.int32)

#using the maximum y value get the bottom left and bottom right points. x = (y - c)/ m
bottom_left_point = np.array([((avg_max_y - avg_left_c) / avg_left_m), avg_max_y], dtype=np.int32)
bottom_right_point = np.array([((avg_max_y - avg_right_c) / avg_right_m), avg_max_y], dtype=np.int32)
```
Use the points to draw lines:

```
cv2.line(img, (bottom_left_point[0], bottom_left_point[1]), (top_left_point[0], top_left_point[1]), color, thickness)
cv2.line(img, (bottom_right_point[0], bottom_right_point[1]), (top_right_point[0], top_right_point[1]), color, thickness)
```
    
The resulting image will look like the following:
[@ Extrapolate Hough Lines  | center | img06]: ./examples/extrapolate_hough_lines.png

You can apply slope thresholds to filter out the lines which do not follow the same path as the lanes. Not applying the slope thresholds can otherwise result in bugs :

[@ Incorrect slopes  | center | img07]: ./issues/bug.png

Once the lines have been drawn, add to the copy of original image to get results like:

[@ Hough Lines on original image  | left | img08]: ./examples/solidWhiteCurve_hough.jpg
[@ Extrapolate Hough Lines  on original image | right | img09]: ./examples/solidWhiteCurve_final.jpg


###2. Potential shortcomings 
With the current approach the following shortcomings are observed :
* The current approach assumes the road to be free of any shadows : This was something that was observed while working on *challenge.mp4* video. The shadows results in a darker lane which may probably match with the road. The thresholds for canny have to be reduced and to avoid noises Gaussian kernel value and along with Hough Transform parameters needs to be adjusted.
* Issues when the lane color and road colors are almost same. Again this was observed while working on *challenge.mp4* around the cement patch.
* May not work in bad weather. 
* Currently it assumes averaging the lanes lines based on the certain threshold history points which may change with the speed of the vehicle and the terrain.
* Turns / curves on road detection will not work well.
* zebra crossing and speed bump marking may confuse the pipeline. 

###3. Suggest possible improvements 
Following improvements needs to be tested and tried :
* Need to remove shadows. Was exploring BackgroundSubtractor (cv2.createBackgroundSubtractorMOG2()) for the same. I could be completely wrong here.
* There has to a feedback mechanism to adjust the thresholds for the frame depending the  weather, road, etc..
* Turns and curve detection, may require map integration.

---
##Results
The results of the project are stored in "final" folder. The folder includes the following:
* extra.mp4	
* white.mp4
* yellow.mp4
* solidWhiteRight	
[@ solidWhiteRight_hough.jpg | left | img10]: ../final/solidWhiteRight_hough.jpg
[@ solidWhiteRight_final.jpg | right | img11]: ../final/solidWhiteRight_final.jpg
* whiteCarLaneSwitch
[@ whiteCarLaneSwitch_hough.jpg | left | img12]: ../final/whiteCarLaneSwitch_hough.jpg
[@ whiteCarLaneSwitch_final.jpg | right | img13]: ../final/whiteCarLaneSwitch_final.jpg
* solidYellowLeft
[@ solidYellowLeft_hough.jpg | left | img14]: ../final/solidYellowLeft_hough.jpg
[@ solidYellowLeft_final.jpg | right | img15]: ../final/solidYellowLeft_final.jpg
* solidWhiteCurve	
[@solidWhiteCurve_hough.jpg | left | img16]: ../final/solidWhiteCurve_hough.jpg	
[@ solidWhiteCurve_final.jpg	 | right | img17]: ../final/solidWhiteCurve_final.jpg	
* solidWhiteRight	
[@ solidWhiteRight_hough.jpg | left | img18]: ../final/solidWhiteRight_hough.jpg
[@ solidWhiteRight_final.jpg | right | img19]: ../final/solidWhiteRight_final.jpg
* solidYellowCurve
[@ solidYellowCurve_hough.jpg | left | img20]: ../final/solidYellowCurve_hough.jpg
[@ solidYellowCurve_final.jpg | right | img21]: ../final/solidYellowCurve_final.jpg
* solidYellowCurve2
[@ solidYellowCurve2_hough.jpg | right | img22]: ../final/solidYellowCurve2_hough.jpg
[@ solidYellowCurve2_final.jpg| right | img23]: ../final/solidYellowCurve2_final.jpg