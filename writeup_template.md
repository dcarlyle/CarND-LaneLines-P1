# **Finding Lane Lines on the Road** 

---
Humans use their eyes to decide where to go. The lines on the road that show where the lanes are and act as a constant reference for where to steer the vehicle.

This project develops an algorithm in a series of steps (pipeline) to:
* Making lane lines clearer to distinguish from their surroundings
* Identify lane lines
* Identify their direction (left or right and angle)

The solution was developed in Python primarily utilising the libraries openCV and numpy 
---

# Reflection
The analysis was broken into the following phases:
- **White lanes** - Identify white lane markings on the right side of the road and broken white markings on the left
- **Yellow lanes** - Identify yellow lane markings on the left side of the road
- **Curvatures** - Identifying a bend in the road 

Each of the above applied an additional step or refinement to improve the pipeline from the prior analysis.

## Pipeline modifications
### Initial pipeline
The initial pipeline was built to identify lane lines as viewed from the a driver’s perspective through the main front car windscreen.

![image of a normal road](https://github.com/dcarlyle/CarND-LaneLines-P1/blob/master/test_images/solidWhiteCurve.jpg "image of a normal road")

The steps in this pipline were:
* *Gray scale* - improving contrast
* *Gaussian smoothing* - to suppress noise and spurious gradients by averaging them out
* *Canny edge detection* - for detecting strong edges
* *Mask* - only the area of the road with road markings needs to be analysed
* *Hough lines* - convert image from *image space* to *parameter space* 
* *Draw lines * - plotted each line segment identified in the mask region onto the image

*Note* Hough transforms a line in image space into a single point in parameter space (Hough space), Parallel lines are represented by two points at the same slope (m) value, but different b (intersections) values. Additional information can be found at [Understanding Hough Transform With Python](https://alyssaq.github.io/2014/understanding-hough-transform/)
OpenCV provides  a slightly faster variant of Hough Transformation, which incorporates probability, HoughLinesP.

![pipeline](https://raw.githubusercontent.com/dcarlyle/CarND-LaneLines-P1/master/images/pipeline.png "pipeline")

#### Limitations
The Draw_Lines function is able to enhance the road markings by painting each line it detects in red on top of the image. Forming a clear indication of the location of the lines.

However on the left side of the image, the broken lane lines are not continuous. The current pipeline gives no predictions mathematically on the *average* direction.

### White lanes
The while lanes analysis, created the pipeline function *process_image* with which to process the each image.

The significant modification was to the draw_lines function, from simply printing the multitude of lines detected. The function separated the lines into left and right, defined by their slope.  **Note** this was only possible as the video footage was from a car on a straight road.

Additional separation was included to identify vertical and horizontal road markings, as these should not be present in a car travelling on a straight road.

The average slope (**m**) and intersection (**b**) was calculated from the set of right lines. This was repeated for the set of left lanes.

From this a single line on the right of the image could be drawn at the correct slope, originating at the bottom of the image (**max y**)  to the apex of the masked area.

This was repeated for both sides, giving a continuous line, which hopefully masks the actual road markings and provides the details of the actual direction of the road.

### Yellow lanes
Applying the same pipeline to video footage with a yellow continuous left lane did not produce consistent left and right lane indicators, as shown:

![white lane pipeline on yellow lanes](https://raw.githubusercontent.com/dcarlyle/CarND-LaneLines-P1/master/images/white_on_yellow.png "white on yellow pippeline")

It should not have failed on the broken white lines on the right, but it did. The left yellow line markings only appeared to be marked some for the time by the pipeline.

The following two improvements were applied:
* **lower mask’s horizon** - to remove the contamination of the left and right line sets, introduced by the curvature at the end of the road markings (those closest to the horizon). Unfortunately the curvature introduced a change in the slope, the slope was being used to distinguish left and right lane markings. This contamination led to the averaging of the slope containing lane markings from other lanes (*e.g. the tip of the right lanes being placed in the set of left lanes*)
* **introduce colour filters** - using the openCV Hue Saturation and Light (HSL) filter to the image in a new function called *filter_HSL* allowed the yellow lane lines to be more easily distinguished. Recombining the new image the white lane lines produced clearer lane lines on both sides of the image.

![HSL](https://raw.githubusercontent.com/dcarlyle/CarND-LaneLines-P1/master/images/HSL_filter.png "HSL filter on yellow lane")

## Pipeline shortcomings 
The pipline only has two parts in essence:

1. Enhance the lane lines
2. Detect the direction of the lanes lines

### Lane line visibility
There are some additional filters that could be applied to enhance the lane lines further. However the lane lines are very clear in the image, much clearer than you would find on a country lane or one where the road requires a little maintenance. So the current method is only suitable for clear road markings which are found on freeways, highways and motorways.

### Lane direction
The algorithm in the pipeline above only improved the visibility, however, if a bend in the lane occurs, then the algorithm fails to calculate the slope of the line correctly. The algorithm uses the slope to distinguish between left and right lane markings, the curve causes parts of the lane to be incorrectly qualified.

### Processing time
The algorithm currently computes quite slowly. If we want to know where to to drive before we arrive then, it needs to be as fast (at a minimum the time it takes to drive from the current location in the image to the horizon in a single image).  

## Possible improvements to the pipeline
### Lane line visibility
Additional filters could be used to highlight the edge of the road, these could be used for off road driving.

### Lane direction
A very simple approach was applied in the final rewrite of process_image. This simply split the mask into *right* and *left* masks. This instantly groups the right lanes and left lanes by image area rather than the collection of calculated slopes. 

By having a collection of slopes that make up a left lane, we can then split these by their slope. This allows us to see if a bend in the road is occuring when we start to collect lines with a negative slope. We can even use this to quickly tell when the road is bending sharply or changing direction.

