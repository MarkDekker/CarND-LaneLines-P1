# **Finding Lane Lines on the Road** 

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./md_assets/grey_red_comparison.jpg] "A comparison between the red channel and grayscale for identifying yellow road markings."
[image2]: ./md_assets/grey_red_comparison.jpg] "A comparison between the red channel and grayscale for identifying yellow road markings."

---

## **Reflection**

### **1. Description of the Image Processing Pipeline.**

My pipeline consisted of 6 steps:
  - First, I extracted the red channel from the images. The original approach was to convert the images to grayscale. However, in this approach the yellow road markings are difficult to identify. The yellow/orange road markings have a strong red component though, so they appear very clearly here. The image below illustrates this clearly:

  ![A comparison between the red channel and grayscale for identifying yellow road markings.][image1]

   - The second step is to apply a gaussian blur to the image.  This appears to reduce the number of edges of textures that are detected by the canny edge detection later.  Increasing the blur reduces the noise from textures but also the amount of lines that are identified, so a good blance had to be found here.
   - The next step was to apply canny edge detection to the blurred image.  This isolated the pixels that are in regions with high colour gradients.
   - The results from the edge detection are then masked to focus on the region of interest, namely the road ahead. The masking is very specific to the focal length and orientation of the camera.
   - Hough lines are then extracted and drawn for the respective image.  This routine was extended to produce two clear lines on either side.
   - Finally, the output is blurred slightly and merged with the original image as an output.

In order to draw a single line on the left and right lanes, I modified the draw_lines() function by adding a line filtering option. This calls a routine that is meant to process the array of lines that are produced by cv2.HoughLinesP to return two lines, one for each road marking. Two approaches were pursued for this filtering function:
 - The first `average_relevant_lines()`, filters out all relevant lines from the set by checking whether they have a feasible slope (i.e. it is not too shallow) and then it averages all the lines passing through the right bottom half of the image with one another and then repeats this for all lines passing through the left half. Another option here would have been to average all lines with a positive slope and then repeat this for all lines with a negative slope.
 - The second approach `select_relevant_lines()`, and probably the more robust method is to select the two lines whose bottom intercept lies closest to the centre of the image. This would ensure that if a parallel line in another lane is detected that this does not have an influence on the positioning of the current lane lines. Below is a comparison of the methods for such a situation:

 ![A comparison between `average_relevant_lines()` on the left and `select_relevant_lines()` when lane markings from a parallel lane are detected.][image1]

 For both approaches, the resulting two lines are extrapolated from the top of the image to the bottom and then masked again with the original region mask.


### **2. Identify potential shortcomings with your current pipeline**

There are a variety of shortcomings with the current pipeline, but also the approach for detecting the lane markings:

 - The main problem is that the approach does not seem very robust. There are countless edge cases for which this approach would fail, even with more extensive finetuning. Examples for this are lane changes, construction sites, unclear roadmarkings (which are surprisingly common), road junctions (especially angled lane merging). Similarly, if wet weather causes many reflections on the road or snow covers large portions of the road, detecting lane markings like this would be difficult. 

 - Besides this the current pipeline results in a lot of jitter for the detected lines between video frames. The effect is especially strong for the striped markings which constanly produce lines with different slopes. This is exacerbated in turns when the markings are curves as opposed to straight lines. 

- Another shortcoming is that the current implementation feels relatively slow.  For a modestly sized video with roughly 500'000 pixels per frame the detection takes relatively long. What would happen for ultra high definition video streams with 20x more pixels per frame?

- The final shortcoming is how this technique works under differing light/exposure. When the colours, contrast and brightness of the input image varies wildly, it is unlikely that the current pipeline will work reliably. 
 
 It it would probably be more sensible to detect the extents of the actual road surface and then identify the road markings on it more 'intelligently', with the appropriate contextual information, such as whether they apply to the current lane, a crossing road or a parallel lane. If each lane marking is identified individually it would also be possible to fit curves though them to map them accurately.

### **3. Suggest possible improvements to your pipeline**

- I believe that further parameter finetuning for the hough lines and canny edge detection would still yield improvements in the consistency with which the lines are detected.

 - Averaging lines from one video frame to the next would also be a useful tool to get more consistent lines and a more robust result.

 - Currently, the allowed slope for the lines detected was used to filter out the edges in the road surface for the most challenging video, this is not satisfactory though. There are cases where these patches of differing road textures have deges running parallel to the lane lines.  Instead, it makes sense to use colour information to filter them out more robustly. 

 - Saving and tracking lines from neighbouring lanes can also be useful at a later stage, especially in positioning the other cars relative to this one and dealing with lane changes.


