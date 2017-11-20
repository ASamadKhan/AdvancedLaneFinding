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

[image1]: ./output_images/ChessImagesForCalibration.png "Undistorted"
[image2]: ./output_images/UndistortedChessImage.png "Chess_Unidistort"
[image3]: ./output_images/ExampleUndistortedImg.png "Road Transformed"
[image4]: ./output_images/unwrapped.png "Perspective Transform"
[image5]: ./output_images/ThreeColorChanels.png "ColorSpaces"
[image6]: ./output_images/LChannel.png "LChanel"
[image7]: ./output_images/BChanel.png "BChanel"
[image8]: ./output_images/PipelineOnImages.png "PipeLine"
[image9]: ./output_images/polyfit.png "Polyfit"
[image10]: ./output_images/histogram.png "Polyfit"
[image11]: ./output_images/polyfit2.png "Polyfit"
[video10]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in mai folder with name AdvanceLaneFinding.ipynb".

An input comprises of number of chessboard images taken with the same camera from different angles. The chessboard has (9,6) corner points. For each chessboard image corners are detected using `findChessboardCorner` cv2 function. The detected image corners are added into **imgpoints** list, which are in fact the pixel locations of chessboard corners for each image. The corners are drawn and siplayed in the below figure usig `drawChessboardCorners` function.
![alt text][image1]
an `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 
![alt text][image2]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

The image below shows the results of applying undistort to one of the project test image:
![alt text][image3]
The undistort is more clear from the corners and the hood of the car.
#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image but it was not working well for me. below image shows different color channels for 3 color spaces.

![alt text][image5]
I chose to use just the L channel of the HLS color space to isolate white lines as shown in the below image
![alt text][image6]
I Used  B channel of the LAB colorspace to isolate yellow lines as shown in the below image.
![alt text][image7]
Below are the result of applying pipeline to test images 
![alt text][image8]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in 4th and 5th code cell of `AdvanceLaneFinding.ipynb` .  The `warper()` function takes as inputs an image (`img`), source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
h,w = undistorted_exImg.shape[:2]

# define source and destination points for transform
src = np.float32([(575,464),
                  (707,464), 
                  (258,682), 
                  (1049,682)])
dst = np.float32([(450,0),
                  (w-450,0),
                  (450,h),
                  (w-450,h)])
```

 I also choose the points programmatically but these points at the end worked with an assumption that that the camera position will remain constant and that the road in the videos will remain relatively flat.
 This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 575, 464      | 450, 0        | 
| 707, 464      | w-450, 0      |
| 258, 682     | 450, h      |
| 1049, 682      | w-450, h        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Jupyter notebook with functions "sliding_window_polyfit" and "polyfit_using_prev_fit" identifies lane lines and fit a second order polynomial to both right and left lane lines. an histogram of the bottom half of the image and finds the bottom-most x position (or "base") of the left and right lane lines. I changed into quarters of the histogram just left and right of the midpoint. This helped to reject lines from adjacent lanes. The function then identifies ten windows from which to identify lane pixels, each one centered on the midpoint of the pixels from the window below. This effectively "follows" the lane lines up to the top of the binary image, and speeds processing by only searching for activated pixels over a small portion of the image. Pixels belonging to each lane line are identified and the Numpy polyfit() method fits a second order polynomial to each set of pixels. The image below demonstrates how this process works:

![alt text][image9]
histogram with the two peaks nearest the center are visible from below image
![alt text][image10]
 using `polyfit_using_prev_fit` I generated the below image. The green shaded area is the range from the previous fit, and the yellow lines and red and blue pixels are from the current image
![alt text][image11]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
