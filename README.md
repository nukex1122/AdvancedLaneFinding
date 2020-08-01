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

[image1]: ./output_images/undistorted_image.jpg "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./output_images/binary_img.jpg "Binary Example"
[image4]: ./output_images/warped.jpg "Warp Example"
[image5]: ./output_images/fit1.jpg "Windowing Technique"
[image5_1]: ./output_images/fit2.jpg "Look Ahead Technique"
[image6]: ./output_images/test1.jpg "Output"
[video1]: ./project_video_edited.mp4 "Video"

### Camera Calibration

The code for this step is contained in the third code cell of the IPython notebook located in `./P2_clean.ipynb`

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. 
To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. 

I used a combination of color, gradient and magnitude thresholds  to generate a binary image at cell 3 in the same file.  Here's an example of my output for this step. 

![alt text][image3]

#### 3. 

The code for my perspective transform includes a function called `getWarpedImage()`, which can be found in the `Pipeline` class in the same notebook.  The `warper()` function takes as inputs an image (`img`) and outputs a birdeye view of the ground. I have hardcoded the source and destination points as follows:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 1280, 700      | 1280, 720        | 
| 0, 700      | 0, 720      |
| 546, 460     | 0, 720      |
| 732, 460      | 1280, 0        |


I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. 

I used the binary image and applied warp transform to it. Afterwards, I tried to fit some polynomials in the image. To fit the polynomials I used two technniques: 

* Windowing Technique: I identified individual starting points for the line and made a list of indices which were in the window. After I got the list, I fitted the lines using `np.polyfit()` function and saved the coefficients. 

* Look Ahead Technique: If I knew what my previous lines were, then I don't need to use the windowing technique again as it is computationally heavier. Instead, I could at the points around my previous fit and make new fits accordinly.

![alt text][image5]
![alt text][image5_1]

#### 5.

Radius of curvature is calculated via `getRadiusOfCurvature()` function which uses the formula $\sqrt{1+(2Ay + B)^2}/(2A)$
where A, B, C are the means of the coefficients of the recent fits of the polynomial $Ay^2+By+C$ 
ROC is also stored in another list, and the mean ROC of the past 20 frames are outputted 


#### 6. 

I implemented this step in the `process()` function of the `Pipeline` class in my code in the same file. Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_edited.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

* The lines and the changes to ROC weren't smooth, so a smoothening process was applied to it. 
* A huge time was spent to configure the parameters so that it is able to work on most frames, yet however, in the challenge video and the harder challenge video, it fails to identifies the correct lines.
* This pipeline fails in the presence of shadows and the polyfit polynomial starts including those points too, thus outputing a very absurd polynomial.
* This pipelin also fails when the camera isn't clean, the brightness changes very randomly and there are other obstacles near your lanes which are getting identified as lanes instead of the lanes itself, thus again, outputting a very absurd polynomial.
* More time could be spent in tuning the parameters, especially the masks by checking out various color spaces and their combinations to make the pipeline more robust
* Some ML approaches could be employed which could identify the lanes
