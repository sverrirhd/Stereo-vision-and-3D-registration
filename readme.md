# Cloudstitch
## Stereo vision - 3D point cloud generation and registration

This project describes my attempt at creating a tool that uses two smartphones to generate and combine multiple 3D point clouds using OpenCV


## Steps
### 1.  Design and 3D print a mount for two smartphones
One of the first things I had to do was the design and 3D print a rig to hold the two smartphones in place. The reason for using a rig like this was to maintain the relative positions of the two cameras between photos so that rectification and camera parameters would only need to be found once and could be reused each time. 

Design             |  3D print
:-------------------------:|:-------------------------:
<img src="https://github.com/sverrirhd/Stereo-vision-and-3D-registration/raw/main/Images/3D%20design.png" alt="drawing" width="450"/>  |  <img src="https://github.com/sverrirhd/Stereo-vision-and-3D-registration/raw/main/Images/Printed.png" alt="drawing" width="450"/>


### 2. Calibrate with a checkerboard to extract rectification and camera parameters
This step used a built-in calibration tool from Matlab that took in a set of photo pairs from the two cameras and outputted parameters that could be used for rectification. 
<img src="https://github.com/sverrirhd/Stereo-vision-and-3D-registration/raw/main/Images/Calibration.png" alt="drawing" width="900"/>




### 3. Image rectification
The parameters from this step can be used to project the images onto imaginary coplanar planes that have desirable computational properties for estimating 3D coordinates of points in the image. 
![image](https://user-images.githubusercontent.com/35537164/162450755-a16556ba-6bd5-4402-9f6f-2985a8d6fedd.png)

"Rectification ... reduces the problem to the epipolar geometry produced by a pair of identical cameras placed side by side with their principal axes parallel" (Hartley and Zisserman, Multiple View Geometry, 2003)

1. Project the images onto new coplanar image planes
2. Project the epipole in the left image to the point (1,0,0)T at infinity 
3. Find the corresponding projection for the right image. This still leaves some unrestrained parameters so we fix the centre point in one image and we now have a fully constrained system.

Once this is done, each pixel in the two images should be located in exactly the same y-coordinates such that triangulation can be done with only the disparity (or translation in image coordinates)

### 3. Extra (Uncalibrated rectification using RANSAC)
While rectification with the learned parameters of the stereoscopic setup is mathematically uncomplicated, it's not very robust due to how sensitive this calibration is to minute changes in the camera rig. Due to this problem I looked into an alternative approach for rectification using a method known as RANSAC (Random sample consensus). 

1. The first step is to find matching points between the two images. This is done using SIFT (Scale-invariant feature transform) features and then by finding nearby points in the new featurespace. There are many slight alterations to this approach which can be used to get slightly more robust matches. 
2. The second step is to use RANSAC to compute the fundamental matrix with the matched points. A fundamental matrix is a matrix that describes a relationship between any two images of the same scene that constrains where the projection of points from the scene can occur in both images. To compute a fundamental matrix, 8 matching points are needed. 
    1. The RANSAC method is applied here by randomly picking 8 matches (point correspondences)
    2. Compute fundamental matrix with 8 point algorithm
    3. Project all points with fundamental matrix
    4. Count number of inliers (i.e. number of points that now match in both the feature space and in pixel coordinates)
    5. Return the matrix with most inliers
3. Find the epipoles $F_p = 0$ and $p^{T} F = 0$
4. Do same as with calibrated
![image](https://user-images.githubusercontent.com/35537164/162452975-b9b679c8-66c9-47ed-8d1b-30b03b764fd3.png)

### 4. Use rectified images to compute the disparity map
To generate 3D points from 2D images one has to perform triangulation. After rectification, the location of two points in this triangle is known and the relative coordinates of the third point in the triangle are found using the disparity. The disparity is the translation of a point in image coordinates (along the x-axis)

![image](https://user-images.githubusercontent.com/35537164/162451604-9bb0d8a4-f574-44d7-8ce4-f829106c42b1.png)

### 5. Generate 3D point clouds

![image](https://user-images.githubusercontent.com/35537164/162455021-5e9f49d2-9b49-4939-a164-7cb920283bc3.png)


### 6. Stitch together the 3D pointclouds

![image](https://user-images.githubusercontent.com/35537164/162455101-65e7878a-3c2f-4e38-b607-0b487628a8c7.png)


![image](https://i.imgur.com/bthF7PX.gif)
