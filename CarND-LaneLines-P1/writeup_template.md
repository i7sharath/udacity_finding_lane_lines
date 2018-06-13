# **Finding Lane Lines on the Road** 

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 5 steps. 
1) First , I converted the the image into grayscale . 
2) Next I applied a gaussian blur with a kernel of size 7 
3) Then I applied canny edge function over the image with a low, hgih threshold of 50 and 150 
4) I applied an ROI too after covering a most-probable area of where our lane lines will be. 
4) Then I called the `hough_lines` function with the following parameters
      - rho = 1
      - theta = 1 deg
      - threshold = 20
      - min_line_len = 20 
      - max_line_gap = = 300
      
      The paramters were tweaked and tuned to find out what works best given my algorithm , however , they may not be optimum
 5) Now so far , I just followed the steps of the computer vision basics to get the pipeline ready 
 6) Then I worked on the draw_lines function which gets called within the Hough Lines , so lets talk a little about that . 
 
 
**Draw Lines**

The objective was to basiclly get an extrapolated long smooth solid line that closely follows the lane lines <br/>
To get it to work there are few aspects to it and I shall explain them in the order I discovered them and it goes a bit above the scope of the function too 

* Averaging and Extrapolating 

1) So first things first , its identified that we need to work on two sets of lines , left lane and right lane
2) So first step is separating them , I used the slope and x-intercept to put a quick filter to get them separated . 
      - This not just separates them into two lines but also additionally filters noisy lines within this ROI 
      - I used x intercept to make thigs simple since the Y axis is reversed in image
1) Took all the `x-points` and `y-points` of all lines of interest and used the `Polyfit` function in numpy 
1) The polyfit , fits a line and gives us the slope and intercept
1) Then I made a line out of the above using the `poly1d` function  . Thanks numpy !
1) Interpolation and Extrapolation
      - So that we have a few more points , this step is actually redundant , but I had a future use so I included 
      - Basically linearly interpolates between start and end with a given number of points using `linspace` function
      - Finally make a list of all points and take the extremities of the these points 
      - And fit a line to it = extrapolation . Voila this is our first version of line . More stuff will happen
      
3) Another aspect is more like bookkeeping , that was crucial to make sure , we dealt with cases when we dont have lines 
      - Basically some image frames , we get nothing significant 
      - So for that we take the past frame as a reference 
      
4) Storing and using past Frames 
      - This is an important aspect since this makes the rendering smooth and handles tricky areas
      - So basically I made two deques , one to handle left lane and other for right lane 
      - They are `left_queue` and `right_queue` with a variable buffer size. After playing with it , I went for 120
      - The main application owns the memory for the queue and we keep appending to it 
      - When we process a new frame , we make a series of checks and we find the best fit line given all points in the frame , then we add this information to the line queue and do a `polyfit` line fit again to get another line 
      - Technically we could do a simple average here too . Tried both , subtle differences , needs more testing though. 
      - Also if we dont like information in the current frame , we default to previous frame 
      - This means we never have blinking lanes ever 
      
 



![alt text][image1]


### 2. Identify potential shortcomings with your current pipeline

**Software shortcomings**
1) Many paramters and hand tuning them is tedious and not feasible 
2) I didn't have time to make it more OOP style , potentially I would do that for sure 
3) Create a config and have tunable parameters
4) Test cases and Benchmarking not present yet 


**Algorithm Shortcoming** 
1) Doesn't handle all cases , hard to make generic
2) Currently needs more work to handle multiple lanes more than 2 in the ROI 
3) Too much noise and relies on the reflectivity of the surface
4) Performance will degrade with quality of lane lines 
5) Night time or flashy sun might give different results , or may need more preprocessing 


### 3. Suggest possible improvements to your pipeline

1) All the above software shortcomings can be addressed and implemented 
2) We can use pregenerated models for lane lines with different reflectivity and lane width for additional paramteres
3) Ofcourse we need training and a neural network classifier for lanes would be great given the same pipeline to replace the Hough Transform

