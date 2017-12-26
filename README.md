

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
[image12]: ./output_images/drawlane.png "drawlane"
[image13]: ./output_images/draw_data.png "draw_data"
[video10]: ./output_images/project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points



### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in main folder with name AdvanceLaneFindingExplaination..ipynb".

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
below image shows R,G,B chanels for RGB colorspace and H,L,S channels for HLS colorspace..
![alt text][image5]
I used a combination of color and gradient thresholds to generate a binary image . For me R-channel combined with gradient and S-channel with gradient worked very well.  
Below image shows the result when R-channel with gradient is applied
![alt text][image6]
Similarly the below image resulted when S-channel gradient is applied..

![alt text][image7]
descroption of pipeline
*Input image will be undistorted using the prameters calcuated from calibration
*Perspective transformed applied to the undistorted image
*L-channel gradiet binary image calculated from transformed image
*S-channel gradient binary image calculated from transformed image
*Both binary images are combined to create a combined image
The below image is resulted after applyig the pipeline to test images.
![alt text][image8]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in 4th and 5th code cell of `AdvanceLaneFindingExplaination.ipynb` .  The `warper()` function takes as inputs an image (`img`), source (`src`) and destination (`dst`) points.  after experimentation I manually selected the 'src' and 'dst' points.

```python
# define source and destination points for transform
src = np.float32([[  595,  450 ],
       [  685,  450 ],
       [ 1000,  660 ],
       [  280,  660 ]])
dst = np.float32([[  300,  0 ],
       [  980,  0 ],
       [ 980,  720 ],
       [  300,  720 ]])
```

 The src and dst are selected with an assumption that that the camera position will remain constant and that the road in the videos will remain relatively flat.
 This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 595, 450      | 300, 0        | 
| 685, 450      | 980, 0      |
| 1000, 660     | 980, 720      |
| 280, 660      | 300, 720        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Jupyter notebook with functions "polyfit" and "previous_fit" identifies lane lines and fit a second order polynomial to both right and left lane lines. an histogram of the bottom half of the image and finds the bottom-most x position (or "base") of the left and right lane lines. The function then identifies ten windows from which to identify lane pixels, each one centered on the midpoint of the pixels from the window below. This effectively "follows" the lane lines up to the top of the binary image, and speeds processing by only searching for activated pixels over a small portion of the image. Pixels belonging to each lane line are identified and the Numpy polyfit() method fits a second order polynomial to each set of pixels. The image below demonstrates how this process works:

![alt text][image9]
<br />histogram with the two peaks nearest the center are visible from below image<br />
![alt text][image10]
 <br />using `previous_fit` I generated the below image. The green shaded area is the range from the previous fit, and the yellow lines and red and blue pixels are from the current image<br />
![alt text][image11]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The code for the raduis of curvature of the lane is in cell titled "calc_curv_rad_and_center_dist" using this line of code (The overall logic):
` left_curverad = ((1 + (2*left_fit_cr[0]*y_eval*ym_per_pix + left_fit_cr[1])**2)**1.5) / np.absolute(2*left_fit_cr[0])`
   ` right_curverad = ((1 + (2*right_fit_cr[0]*y_eval*ym_per_pix + right_fit_cr[1])**2)**1.5) / np.absolute(2*right_fit_cr[0])`
The position of the vehicle with respect to the center of the lane is calculated with the following lines of code:<br />

` car = binary_warped.shape[1]/2
  left_fit_line = l_fit[0]*h**2 + l_fit[1]*h + l_fit[2]
  right_fit_line = r_fit[0]*h**2 + r_fit[1]*h + r_fit[2]
  center_lane = (left_fit_line + right_fit_line) /2
  center = (car - center_lane) * xm_per_pix
    `

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

`drawlane` function Draw Curvature Radius and Distance from Center Data onto the Original Image in the Jupyter notebook.  The image below is an example of the results of the draw_lane function:

![alt text][image12]
<br />`draw_data` function, which writes text identifying the curvature radius and vehicle position data onto the original image:<br />
![alt_text][image13]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_images/project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The problems I encountered were almost exclusively due to lighting conditions, shadows, discoloration. I faced a lot of difficulties in this assignment.  
