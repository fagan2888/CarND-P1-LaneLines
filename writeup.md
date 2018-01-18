# **Finding Lane Lines on the Road** 

## Writeup
---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

[image2]: ./examples/edges.jpg "Edges"

[image3]: ./examples/blur.jpg "Blur"

[image4]: ./examples/roil.jpg "ROIl"

[image5]: ./examples/roir.jpg "ROIr"

[image6]: ./examples/bitor.jpg "BitOr"

[image7]: ./examples/result.jpg "Result"
---

### Reflection

### 1. Pipeline Description.


My pipeline consists of the following steps. 

First, I convert the images to grayscale to retain only the intensity information, which is enough for the purpose of edge detection to detect the lane lines. 

![alt text][image1]

Second, I smoothen the image using Gaussian blur to remove noise.

Third, I pass the image through Canny edge detector to obtain the sharp boundaries in the image, which includes the lane lines on the road.

![alt text][image2]

Fourth, I smoothen the obtained edged by passing it through Gaussian filter. This smoothens out sharp noises in the detected edges.

![alt text][image3]

Fifth, Its time to apply a region mask to remove all edges except those of the lane lines. For this, I consider the left lane region and the right lane region separately. This provides better control over the region of interest for the lanes: It prevents the Hough transformation from mixing up the lines between the two edges.

![alt text][image4] ![alt text][image5]

Sixth, I pass each of the two images through the Probabilistic Hough-lines function to obtain the lane lines from the edges in the region of interest. Then I combine the two separate images into a single image using bit-wise-or.

![alt text][image6]


Finally, I  overlay the lane marks by passing the image along with the original image through the weighted_img function.



![alt text][image7]


In order to draw the lane-line marker on the left and right lanes, I modified the `draw_lines()` and `hough_lines()` methods such that only the longest continuous line returned by the Hough function is marked. I also introduce two additional arguments `y0` and `y3` which denote the top and bottom y-coordinates respectively of the extrapolated lane-lines, i.e. how much the lane lines are to be extended in either direction. If the line detected by the Hough function be `[(x1,y1),(x2,y2)]` then this would be extrapolated such that `[(x0,y0),(x1,y1),(x2,y2),(x3,y3)]` be co-linear. 

I create three additional helper methods

1. `get_mc(x1, y1, x2, y2)` that returns the slope `m` and y-intercept `c` for the line `[(x1,y1),(x2,y2)]`

2. `get_line(m, c, y0, y3)` that returns the extrapolated line between `y0` and `y3` for given `m` and `c`

3. `project_longest_lines(img, lines, y0, y3, color=[255, 0, 0], thickness=2)` that overlays the longest `line` in `lines` extrapolating it between `y0` and `y3` if and only if they are tilted at angles whose absolute value is in between 20 degrees and 40 degrees off the vertical. 

The `draw_lines` method is then modified as 
```python
def draw_lines(img, lines, y0, y3, color=[255, 0, 0], thickness=2):
    if len(lines) > 0:
        project_longest_lines(img, lines, y0, y3, color, thickness)
```
### 2. Identify potential shortcomings with your current pipeline

1. Since I've based my lane detection on the longest line among the lines returned by the Hough function, problems may occur in lane detection if there are longer marks just beside the lane marks, such as tyre marks of other vehicles,pavement linings or other kind of shadows.

2. The detected line overlays are not steady and tend to flicker too much, especially when the lane lines are not properly identifiable due to dust cover, shadow or otherwise. This is specially noticeable in the challenge video where the transition takes between a darker road floor to a lighter road floor.

3. It might not perform well in scenarios where the road turns and bends. 

### 3. Suggest possible improvements to your pipeline

1. One way to improve the lane detection would be by trying to detect both left and right edges of each lane line such that it actually denotes a lane line and not some random marks on the road. In that way there will be lesser chances of a false positives in lane-line detection.

2. Perspective may be taken into account for better detection of parallel lane lines.
