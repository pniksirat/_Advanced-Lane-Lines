## Writeup Project 2

---

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

[image1]: ./CarND-Advanced-Lane-Lines/output_images/undistorted_Camera_calibration3.jpg "Corners"
[image2]: ./CarND-Advanced-Lane-Lines/output_images/calibration1.jpg "Distorted_Orignal"
[image3]: ./CarND-Advanced-Lane-Lines/output_images/undistorted_Camera_calibration1.jpg "Undistorted"
[image4]:./CarND-Advanced-Lane-Lines/output_images/Undistorted_test.jpg "Undistorted Road"
[image5]: ./CarND-Advanced-Lane-Lines/output_images/Threshold_test1.jpg "Thresholding"
[image6]: ./CarND-Advanced-Lane-Lines/output_images/bird_eye_test2.jpg "Warped"
[image7]: ./CarND-Advanced-Lane-Lines/output_images/FinalImg_test2.jpg "Final_Image"



## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### The rubric points  and describtion of implementation  

All imgaes output from the code is in ./CarND-Advanced-Lane-Linesoutput_images for reference. 
following functions are used from the quizes in the coarse work :Camera calibration and undistortng, Thersholding, Window searching, warping. 


### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first cell of the IPython notebook located in "./CarND-Advanced-Lane-Lines/p2.ipynb" 
I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image using cv2.findChessboardCorners function.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection. The images of detected corners are availible in
"./CarND-Advanced-Lane-Lines/output_images/", called: WCorners_ImageName

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 


![alt text][image1]
![alt text][image2]
![alt text][image3]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

First step the image of the raod is getting undistorted based on camera matrix and coefficients, ( before feeding through thresholding. 

![alt text][image4]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of sobelx & sobely, magnitude & direction (arctan2(sobely/sobelx)), in addition to HLS (S-channel) thresholds to generate a binary image (thresholding steps in the cell two, section marked as Thresholding).  Here's an example of my output for this step. 

The code is using following limits to better extract the lanes and eliminates shadows:

SobelX & SobelY:(10,220)
magnitude:(50, 220)
direction: (0.8, np.pi/2)
Schannel_HLS:(100, 255)

![alt text][image5]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in lines the second cell of the code, right after thresholding marked as Warped.  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following corner pints:

source :[[600,450], [680,450], [240,720],[1050,720]]

destination :[[350,0],  [930,0], [350,720],[930,720]]

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image6]


#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

For the first polynomial fit, the first window is estimated around the two peaks of the histogram, then the nonzero indices for x and y found for that window. To update the next window check lane_pixels if more than min desired pixels, then recenter next window on their mean position.    
The polynomial fit once found for the next frames, is used for feature detection in away that we search around the same area for nonZero pixels. All the good pixels defined as a lane is saved for later to use for polyfit function. 

The polyfit function outputs a set of coefficients (array of three elements, a,b,c), which then used to find the polynomial line x=ay^2+by+c  


Once the fit identified for the lanes they are passed on to the next LaneDetection_pipline for the consecutive frames; as leftPast and RightPast.  

This function described above is called LaneDetection_Window


#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The raduis of curvature is estimated in a function measure_curvature_meter which estimate the curvature at the point close to the vehicle where is the maximum value of Y in the image; it takes polynomial coefficients and Y plot points that are already scaled (pixel to meter) then based on the curvature formula using first & second polynomial coefficents & y max. 

check the coefficient of the polynomial to see if the estimated polynomial is in fact parallel, and if so then average the lane curvatures.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

At this point the polynomial estimation needs to be warped back to the initial form from the bird_eye view. After the perspective transformation and warpPerspective (using warp function and switching src and dst in the function) the output image can be used. First create an empty matrix given the dimention of colored image to draw the lanes with cv2.polylines, then using cv2.addWeighted calculates the weighted sum of two image arrays of original image and the one with lane lines drawn. 

Here is an example of my result on a test image:

![alt text][image7]


---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./CarND-Advanced-Lane-Lines/output_images/project_video_output.mp4)


### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?


As described in the lectures,there are shadow and birght areas on the road, where using S-Channel of HLS combined with other threshold binarys can improve the detection of those lines with bright patches. The tunning of the thresholding limits was done just by visualizing and manually, there might be a better optimal limits which could improve the results. Also the series of lane lines X polynomial values appended into an array for each frame, to be used for smoothing using the mean of the n last polynomial.
I also noticed that on the video there is a point around 0.42 min that the polynomial detection became somewhat erroneous due to the patchy lines that are disparced with distance. It recover after that frame. However, averaging and tunning the parameters didn't improve the video at that point. I think the issue is that there was a big gap between the first detected lane segment and the next and polynomial priortized the further lines since it was more points identified.  

