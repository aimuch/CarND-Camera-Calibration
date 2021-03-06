## Camera Calibration with OpenCV

The IPython notebook in this repository contains code to calculate the camera matrix and distortion coefficients using the images in the "calibration_wide" folder.

```python
%%HTML
<style> code {background-color : pink !important;} </style>
```


<style> code {background-color : pink !important;} </style>


Camera Calibration with OpenCV
===

### Run the code in the cell below to extract object points and image points for camera calibration.  


```python
import numpy as np
import cv2
import glob
import matplotlib.pyplot as plt
%matplotlib qt

# prepare object points, like (0,0,0), (1,0,0), (2,0,0) ....,(6,5,0)
objp = np.zeros((6*8,3), np.float32)
objp[:,:2] = np.mgrid[0:8, 0:6].T.reshape(-1,2)

# Arrays to store object points and image points from all the images.
objpoints = [] # 3d points in real world space
imgpoints = [] # 2d points in image plane.

# Make a list of calibration images
images = glob.glob('calibration_wide/GO*.jpg')

# Step through the list and search for chessboard corners
for idx, fname in enumerate(images):
    img = cv2.imread(fname)
    gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)

    # Find the chessboard corners
    ret, corners = cv2.findChessboardCorners(gray, (8,6), None)

    # If found, add object points, image points
    if ret == True:
        objpoints.append(objp)
        imgpoints.append(corners)

        # Draw and display the corners
        cv2.drawChessboardCorners(img, (8,6), corners, ret)
        #write_name = 'corners_found'+str(idx)+'.jpg'
        #cv2.imwrite(write_name, img)
        cv2.imshow('img', img)
        cv2.waitKey(500)

cv2.destroyAllWindows()
```
    (1, 2)
    

### If the above cell ran sucessfully, you should now have `objpoints` and `imgpoints` needed for camera calibration.  Run the cell below to calibrate, calculate distortion coefficients, and test undistortion on an image!


```python
import pickle
%matplotlib inline

# Test undistortion on an image
img = cv2.imread('calibration_wide/test_image.jpg')
img_size = (img.shape[1], img.shape[0])

# Do camera calibration given object points and image points
ret, mtx, dist, rvecs, tvecs = cv2.calibrateCamera(objpoints, imgpoints, img_size,None,None)


dst = cv2.undistort(img, mtx, dist, None, mtx)
cv2.imwrite('calibration_wide/test_undist.jpg',dst)

# Save the camera calibration result for later use (we won't worry about rvecs / tvecs)
dist_pickle = {}
dist_pickle["mtx"] = mtx
dist_pickle["dist"] = dist
pickle.dump( dist_pickle, open( "calibration_wide/wide_dist_pickle.p", "wb" ) )
#dst = cv2.cvtColor(dst, cv2.COLOR_BGR2RGB)
# Visualize undistortion
f, (ax1, ax2) = plt.subplots(1, 2, figsize=(20,10))
ax1.imshow(img)
ax1.set_title('Original Image', fontsize=30)
ax2.imshow(dst)
ax2.set_title('Undistorted Image', fontsize=30)
```




    Text(0.5,1,'Undistorted Image')




![png](./readme_img/output_4_1.png)

## Focal length from calibration parameters
Q: 
  I am trying to calibrate my camera (sony alpha 6000 20mm lens) using open CV. In the parameters list, I can find the focal length coefficient as 5192. How can I calculate the corresponding focal length from that ?

A:
You already know the focal lengths is 20mm, but you might want to compare/check calibration results.

In camera matrix the focal lengths `fx`,`fy` are expressed in pixel units. To convert focals in World units Fx, Fy you need sensor size in same units using similar triangle
```
Fx = fx * W /w 
```
or 
```
Fy = fy * H /h
```
where:
W: is the sensor width expressed in world units, let's say mm
w: is the image width expressed in pixel
fx: is the focal length expressed in pixel units (as is in the camera matrix )
Your Sony α6000 is 6000x4000pix and sensor size is 23.5 x 15.6mm hence:
```
Fx = fx * W /w = 5192pix * 23.5mm / 6000pix = 20.33mm
Fy = fy * H /h =  5192pix * 15.6mm / 4000pix = 20.25m
```
At the end your calibration looks good

> https://answers.opencv.org/question/139166/focal-length-from-calibration-parameters/
