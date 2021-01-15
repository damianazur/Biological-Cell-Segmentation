# Biological Cell Segmentation (Cancer Screening)

## Description

The program allows the user to select any image containing biological cells. The program was tested on certival cells as input images.<br>
The algorithm will segment the cells and the user can click on a cell to crop it out of the image and enlarge it.

<img src="https://i.imgur.com/iYTapQk.png" width="100%" height="100%"><br>

## The Algorithm
The algorithm is broken up into 3 stages/sections. The pre-processing stage, the segmentation stage, and finally the image selection stage.
I will be discussing these below.<br>

### Image Pre-processing
To summarize the pre-processing we get the region of interest, get the edges, filter the edges, and put the edges on top of the region of interest.<br>
<br>
1\. The image is converted to the various colour spaces such as RGB and Gray. The gray image is what will be processed and<br>&nbsp;&nbsp;&nbsp;
    the colour image will be used for display purposes.<br>
2\. We obtain the ROI of all the cells with a threshold, the threshold value is calculated by adding the mean value and the standard<br>&nbsp;&nbsp;&nbsp; 
    deviation of the pixels.<br>
3\. The cells ROI noise is removed with Morphological Ex Open and any pockets within the ROI are filled in/closed with the<br>&nbsp;&nbsp;&nbsp; 
    Morphological Ex Close.<br>
4\. Canny edge detection is used to obtain the edges on the image. These edges outline the cells, however there is noise which needs<br>&nbsp;&nbsp;&nbsp;
    to be removed/filtered.<br>
5\. Contours of the edges of the canny edge detection image from the previous step are obtained and filtered by size. This removed <br>&nbsp;&nbsp;&nbsp; the noise edges 
    that were too small and in most cases this also removed the outline of the nucleus which increases the accuracy.<br>
6\. The final result is a binary image with the region of interest in white on a black background with black lines within the region of <br>&nbsp;&nbsp;&nbsp; 
    interst that roughly separating the cells.

### Segmentation
1\. We use the image that has been pre-processed and then dilate it with a 3x3 kernel. This will give us the "Sure background" iamge.<br>&nbsp;&nbsp;&nbsp; 
    The black within that region is definitely the background and this image will be used in the calculation of the "unknown" region that<br>&nbsp;&nbsp;&nbsp; 
    I will explain in the following steps.<br>
2\. We use the distanceTransform() method which produces a gray level image which intensifies the foreground region.<br>&nbsp;&nbsp;&nbsp;
    The pixels closer to the boundary are darker and the further away pixels from the boundary are lighter in shade. This allows us<br>&nbsp;&nbsp;&nbsp; 
    to obtain the "center" of a the cells.<br>
3\. The result of the distanceTransform() is thresholded to give us the "sure foreground". By using the treshold we get the lighter values<br>&nbsp;&nbsp;&nbsp; 
    which we are certain is the foreground as they are the furthest away from the boundary.<br>
4\. We then get the unknown region which lies between the sure background and the sure foreground by subtracting <br>&nbsp;&nbsp;&nbsp; 
    one from the other.<br>
5\. The connectedComponents() method is used to assign a specific shade of gray to each of the sure foreground segments. These gray<br>&nbsp;&nbsp;&nbsp;
    segments are known as markers.<br>
6\. The value 1 is added to each pixel value so that the background colour is not 0. This step is important as we do not want the <br>&nbsp;&nbsp;&nbsp;
    watershed to confuse the background region with the unknown region. It expects the unknown region to be 0, hence why the<br>&nbsp;&nbsp;&nbsp;
    background is of value 1.<br>
7\. Next we take the "markers" image from the previous step and set the pixel value to 0 where the value at the same row and <br>&nbsp;&nbsp;&nbsp; 
    column in the unknown background image is 255. In other words we make the unknown background region on the<br>&nbsp;&nbsp;&nbsp;
    markers image black.<br>&nbsp;&nbsp;&nbsp;
    To recap we now have an image with the background of value 1, the unknown region of value 0 and markers of value greater than 1.<br>
8\. Then we pass the original image and the markers image to the watershed() method and it will assign the unknown region<br>&nbsp;&nbsp;&nbsp; 
    to each marker and that will segment the cells.

### Cell Selection
There are two parts to the cell selection. There is the onclick part and the hover part.<br>
When the user hovers over the cell it will be contoured in red<br>
When a user clicks on a cell it will be isolated, cropped, enlarged by 4 times, saved to the file and displayed to the user<br>
The original window will close.<br>
I decided to go with an image resize factor of 4 because 2 did not seem large enough and I did not want to multiply by an odd number<br>
as that could result in undesired distortion.

#### On Click
1\. The displayImgs and displayTitles variables are used to store the images for the algorithm progress display purposes<br>&nbsp;&nbsp;&nbsp;
    (images for plotting with matplotlib).<br>
2\. If the left mouse button is clicked then we get the colour of the pixel that was clicked on the segmented image.<br>&nbsp;&nbsp;&nbsp; 
    Each segment has their own unique grayscale colour and so we now have a pixel value that uniquely identifies that<br>&nbsp;&nbsp;&nbsp;
    segmented cell<br>
3\. If the pixel that was clicked is of value 1 then it is the background and we do not enhance the image as there is<br>&nbsp;&nbsp;&nbsp; 
    nothing to enhance. Otherwise we go through the next steps to segment and enlarge the image.<br>
4\. Using that pixel value we use a threshold to create a mask that isolates that segmented cell.<br>
5\. Next the algorithm gets a contour of that mask, this contour is sorted to get the largest one at 0th index.<br>&nbsp;&nbsp;&nbsp; 
    This step is not necessary as there should only be one and only one contour but I do it to be safe.<br>
6\. Using the contour we get the bounding box of the cell and we crop the cell out of the image.<br>
7\. We resize the cropped cell image to 4 times it's original size<br>
8\. We save the image to the file<br>
9\. We display the enhanced image to the user on screen and close the previous window<br>
    
#### On Hover
1\. Anytime the user is not clicking the on hover functionality is called. Firstly the pixel colour of the segmented <br>&nbsp;&nbsp;&nbsp;
    shaded region is obtained. Just as with the onClick functionality.<br>
2\. If the user is not hovering over a background pixel (meaning they are hovering over a cell) <br>&nbsp;&nbsp;&nbsp; 
    then we create a mask of the cell using a threshold function and we obtain the contours of the thresholded image.<br>&nbsp;&nbsp;&nbsp;
    Again, this process is the same as with the onClick functionality where we isolate the cell<br>
3\. The contours are drawn in red on top of the copy of the image and displayed to the user. We draw it on top of the copy so that<br>&nbsp;&nbsp;&nbsp; 
    the contours do not stay after the user stops hovering over the cell.<br>

## Conclusion
The program/algorithm has been tested on 7 other images of similar nature and have performed very well on all of them. The results can be viewed in the "Results" folder and the original images are in the main folder. The algorithm is general enough so that it works on similar images. The algorithm is also efficient and runs fast, even when the images are large in size. For example, "Cervical Mono.jpg" contains over a million pixels and still loads very quickly.<br>
The program has the necessary error checking to ensure that it doesn't crash in any scenario. Even when the user has not clicked on an image and exited the program.<br>
The intructions in the beginning simply detail exactly how the program works and the program is very clear and simple.<br>
Contours are drawn around the cell that is hovered over to make the program more transparent and make the user experience enjoyable.<br>
Existing solutions to the problem have been researched and an original solution has been created. The research sources have been cited with the Vancouver citation style.<br>
The progression of the algorithm is displayed with matplotlib in a sequential manner with titles above each image describing the image.


## Research

The research began with looking at research papers that tackle the same problem of segmenting cells with OpenCV. Li *et al.* (2019) **[1]** have investigated the segmentation of cells with OpenCV and introduced me to the use of distance transformation and the watershed algorithm as a means of segmenting cells. Their research has provided me with insight on the topic that served as a starting point for research which led me to create my own solution to the problem.<br>
<br>
After seeing their results I further researched the OpenCV watershed function/algorithm **[2]** to learn about how it works and how it can be applied to the cellular microscopy images. The watershed function works by segmenting the image that contains various region. There are marked regions which represent the center of a cell, there is a background region which is where the cells are definitely not and finally an unkown region which lies between the marked regions and the background. This unkown region is assigned/split amongst each of the markers.
<br><br>
Throughout my research on the watershed algorithm I came across the morhological operations **[3]** that were taught in this module which were crutial in the watershed algorithm. These were useful in many ways which include better defining the region of interest with the use of morpholigical Ex close and open functions which fill in holes in the ROI and remove noise.
The morhological operations which of course involve dilation and/or erosion were used to better define the regions. Regions where we are sure the cell is and where it is not.
<br><br>
I also learned about the OpenCV distanceTransform function **[4]** which is important in the watershed algorithm. The function is used to produce a gray level image which intensifies the foreground region. This shows where the edge of the cell and center of the cell is which plays a great role in the segmentation process.
<br><br>
The connectedComponents OpenCV function **[5]** was used to label each segment with its own unique grayscale value which was very useful in segmenting the cells. This is because we can filter out the clicked region by the pixel value as that pixel value is unique.
<br><br>
Canny edge detection **[6]**, which was taught in the module was useful in determining the edges between the cells and I discovered that it greatly increased the accuracy in the segmentation process by combining the edges with the threshold of the cells. The contours functions, also taught in this module were very helpful in filtering out the edges obtained with Canny edge detection. Filtering the contours by size and ignoring any contours that were too small has improved the effectiveness of the algorithm. The contours functions **[7]** also proved useful for displaying the progress of the algorithm and to higlight the region that will be clicked by the user as to improve the usability aspect of the program.

### References
1\. Li G, Zhang Y, Xu B, Li X. Image Analysis and Processing of Skin Cell Injury Based on OpenCV.<br>&nbsp;&nbsp;&nbsp; 
    Journal of Physics: Conference Series [Internet]. 2019 Jun<br>&nbsp;&nbsp;&nbsp; 
    [cited 2020 Nov 22];1237:032003. Available from: https://iopscience.iop.org/article/10.1088/1742-6596/1237/3/032003<br>
    <br>
2\. OpenCV: Image Segmentation with Watershed Algorithm [Internet]. Opencv.org. 2020<br>&nbsp;&nbsp;&nbsp; 
    [cited 2020 Nov 22]. Available from: https://docs.opencv.org/master/d3/db4/tutorial_py_watershed.html<br>
    <br>
3\. OpenCV: Morphological Transformations [Internet]. Opencv.org. 2020 [cited 2020 Nov 22].<br>&nbsp;&nbsp;&nbsp; 
    Available from: https://docs.opencv.org/master/d9/d61/tutorial_py_morphological_ops.html<br>
    <br>
4\. OpenCV: Miscellaneous Image Transformations [Internet]. Opencv.org. 2020 [cited 2020 Nov 22].<br>&nbsp;&nbsp;&nbsp; 
    Available from: https://docs.opencv.org/master/d7/d1b/group__imgproc__misc.html#ga25c259e7e2fa2ac70de4606ea800f12f<br>
    <br>
5\. OpenCV: Structural Analysis and Shape Descriptors [Internet]. Opencv.org. 2020 [cited 2020 Nov 22].<br>&nbsp;&nbsp;&nbsp; 
    Available from: https://docs.opencv.org/master/d3/dc0/group__imgproc__shape.html#gac2718a64ade63475425558aa669a943a<br>
    <br>
6\. Canny Edge Detection — OpenCV-Python Tutorials 1 documentation [Internet]. Readthedocs.io. 2013 [cited 2020 Nov 22].<br>&nbsp;&nbsp;&nbsp; 
    Available from: https://opencv-python-tutroals.readthedocs.io/en/latest/py_tutorials/py_imgproc/py_canny/py_canny.html<br>
    <br>
7\. OpenCV: Contours : Getting Started [Internet]. Opencv.org. 2020 [cited 2020 Nov 22].<br>&nbsp;&nbsp;&nbsp; 
    Available from: https://docs.opencv.org/3.4/d4/d73/tutorial_py_contours_begin.html<br>
