#**Finding Lane Line on The Road**
##Writeup
---
**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
 
 * Make a pipeline that finds lane lines on the road
 * Reflect on your work in a written report
---
###Reflection
####**1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.**

My pipeline consisted of 5 steps. 

First, I converted the images to grayscale:
```python
def grayscale(img):
    return cv2.cvtColor(img, cv2.COLOR_RGB2GRAY)
gray = grayscale(img)
```

Then I used guassian blur to reduce the noises. The kernel size is 7.
```python
def gaussian_blur(img, kernel_size):
    return cv2.GaussianBlur(img, (kernel_size, kernel_size), 0)
blur_gray = gaussian_blur(gray, 7)
```

The next step is using canny function to find edges of the image. I set the low_threshold and high_threshold to 50 and 150.
```python
def canny(img, low_threshold, high_threshold):
    return cv2.Canny(img, low_threshold, high_threshold)
edges = canny(blur_gray, 50, 150)
```
Then I set the region of interest base on the shape of the image.
```python
imshape = image.shape
vertices = np.array([[(0,imshape[0]),(450, 320), (500, 320), (imshape[1],imshape[0])]], dtype = np.int32)
masked_edges = region_of_interest(edges, vertices)
```
Next step is using hough_lines function to find lines in the region of interest.
```python
line_img = hough_lines(masked_edges, 1, np.pi/180, 15, 5, 10)
```
In the end, combine the original image and lines image together and can get the image like this:
![Image text](https://view54dc5dc0.cn1-udacity-student-workspaces.com/files/CarND-LaneLines-P1/examples/line-segments-example.jpg)
In order to draw a single line on the left and right lines, I modified the draw_lines() function by seperate left lines and right lines. If the slope is greater than 0.5, it should be the right line and if the slope is less than -0.5, it should be the left line.
```python
    left_lane_lines_x = []
    left_lane_lines_y = []
    right_lane_lines_x = []
    right_lane_lines_y = []
    
    for line in lines:
        x1,y1,x2,y2 = line[0]
        if abs(x1-x2) == 0:
            slope = float("inf")
        else:
            slope = (y2-y1)/(x2-x1)
        
        if slope > 0.5:
            right_lane_lines_x.append(x1)
            right_lane_lines_x.append(x2)
            right_lane_lines_y.append(y1)
            right_lane_lines_y.append(y2)
        elif slope < -0.5:
            left_lane_lines_x.append(x1)
            left_lane_lines_x.append(x2)
            left_lane_lines_y.append(y1)
            left_lane_lines_y.append(y2)
```
Then I used np.polyfit to finde the average slope of the left and right line.
```python
    r_m, r_b = np.polyfit(right_lane_lines_x, right_lane_lines_y, 1)
    l_m, l_b = np.polyfit(left_lane_lines_x, left_lane_lines_y, 1)
```
Base on my region of interest and the average slope of the lane line. I can calculate the top and bottom point and draw lines on the image.
```python
    r_x1 = (y1 - r_b) / r_m
    r_x2 = (y2 - r_b) / r_m
    
    l_x1 = (y1 - l_b) / l_m
    l_x2 = (y2 - l_b) / l_m
    
    cv2.line(img, (int(r_x1), y1), (int(r_x2), y2), color, thickness)
    cv2.line(img, (int(l_x1), y1), (int(l_x2), y2), color, thickness)
```
####**2. Identify potential shortcomings with my current pipeline**
There are many shortcomings in my code.

Firstly, my region of interest was fixed. If the camera changed or the view changed, the code will be failed.

Secondly, I can't use this code to pass the challenge. The code shows two crossed lines in the center of the image and shaking very ferquently. I will take this problem to next course and try to find a solution.
###**3. Suggest possible improvement to my pipeline**

A possible improvement would be use another function to reduce the noise. And the region of interest can be changed to a dynamical area that can fit all image shape.

