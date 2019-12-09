## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

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

[image0]: ./project2.gif "Output Gif"
[image1]: ./write_up_images/image1.png "Calibrate Camera"
[image2]: ./write_up_images/image2.png "Undistort road image"
[image3]: ./write_up_images/image3.png "Threshold undistrorted road image"
[image4]: ./write_up_images/image4.png "Transfrom perpectively the thresholded image"
[image5]: ./write_up_images/image5.png "Detect Lines"
[image6]: ./write_up_images/image6.png "Draw Lane"
[image7]: ./write_up_images/image7.png "Warp back the drawn lane image"
[image8]: ./write_up_images/image8.png "Build final image"
[image9]: ./write_up_images/image9.png "Test the pipeline"
[video1]: ./output_project_video.mp4 "Output video"

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

Camera calibration is divided into two small steps:

a) find corners
First operation is to find pattern in chess board. Once we find the corners, we can increase their accuracy using cv.cornerSubPix(). We can also draw the pattern using cv.drawChessboardCorners().

Source: https://docs.opencv.org/3.4/dc/dbb/tutorial_py_calibration.html

The code for this step is contained in the first code cell of the IPython notebook located in "./advanced_lane_Finding.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

b) undistort images
As soon as corners are identified, it is possible to calibrate the camera and undistort the images using the calibration matrix calculated during the calibration.: 

![alt text][image1]

### Pipeline (single images)

#### 1. Apply a distortion correction to road test image

Now that the calibration matrix is available, it is possible to apply it also to some test images
![alt text][image2]

#### 2. Use color transforms, gradients, etc., to create a thresholded binary image

In order to create a satisfying thresholded binary image, color and gradient thresholding are combined. That is because color thresholds (using HLS space) help to detect lane lines of different colors and under different lighting conditions. Gradient thresholds are useful to focus on pixels that are likely to be part of the lane lines. It is possible then to create a binary combination of two different thresholded images to map out where either the color or gradient thresholds were met. (cell 6-7)

![alt text][image3]

#### 3. Apply a perspective transform to rectify binary image ("birds-eye view").

Next, it is necessary to perform perspective transform. This is because of the phenomen of perspective, that let object to appear smaller the further they are from the viewpoint. Perspective applies also to lines, letting parallel lines to appear to converge to a point. A perspective transorm warps the image and effectively drags point towards or away from the viewpoint in order to change the apparent perspective.

In order to do this in the image proposed, first the region of interest need to be exctracted, identifying four source points. Then, the four destination points are defined and the tranformation MAtrix M is calculated. The region of interested will be an isosceles trapezoid, warped into a rectangle. (cells 8-9)

![alt text][image4]

#### 4. Detect lane pixels and fit to find the lane boundary.

At this point, it is necessary to decide explicitly which pixels are part of the lines and which belong to the left line and which belong to the right line.

Plotting a histogram of where the binary activations occur across the image is one potential solution for this. The two highest peaks from the histogram are used a starting point for determining where the lane lines are, and then use sliding windows moving upward in the image (further along the road) to determine where the lane lines go. (cells 10-11)

![alt text][image5]

#### 5. Determine the curvature of the lane and vehicle position with respect to center.

Once the two lines are identified, next step is to calculate the radius of the curvature and the distance from the center of the road. This is a measure of the quality of the steps followe and it can give feedback thinking about real data. The U.S. regulations require a minimum lane width of of 3.7 meters, so knowing the shape of our image we can extract the correspondence between pixels and meters in the image. (cells 12-14)

#### 6. Warp the detected lane boundaries back onto the original image.

It is now time to put all the measures done so far on the original image. Radius of the curve, distance from the center of the lane and the two lines are available. An area representing the lane is displayed together with the indications of Radius and distance. (cells 15-16)

![alt text][image6]

#### 7. Output lane boundaries and numerical estimation of lane curvature and vehicle position.

The lane is then drawn onto the original test image together with the boxes containing road curvature and distance from the center of the road (17-18-19)

![alt text][image7]
![alt text][image8]

#### Complete Pipeline

he pipeline described at the beginning is now used to process the test video and generate the output one. 1) Undistort image. 2) Threshold image 3) Perspective transform 4) Lines finding 5) Lane drawing 6) Warp back the image 7) Text printing
(cells 20-21)

![alt text][image9]

---

### Pipeline (video)

Here is the final video output (./project_video.mp4)

---

### Discussion

Even with the coination of sobel threshold and S channel one, he line detection is reacting with difficult with extreme conditions, like sudden change in the brightness due to tunnel exiting with sun. Weather conditions are then impacting the chain. 
From a performance point of view, the video processing is pretty slow due to high calculations done during the pizels recognition. The whole video is processed in about 30 minutes.

First point can be improved with combining other thresholding methods like meagnitude of the gradient and color masks that fit better.
Second point can be improved by checking if the lane was detected in the previous frame, so that the pixels search can be done only near the already detected one. 
