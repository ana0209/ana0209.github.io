---
layout: post
title: "2D Map Fitting"
date: 2019-10-31
---

This project arose when I was trying to compare several 2D lidar based SLAM algorithms. I was having a robot move around a part of my home with a lidar mounted on top of it. I tested gmapping, Cartographer SLAM and Karto SLAM. I had generated the ground truth map manually. I wanted to fit the calculated maps by each of the algorithms to the ground truth map. It looked simple, but I could not find any ready made, accurate code out there. I found [this project](https://github.com/saeedghsh/Map-Alignment-2) that looks interesting but it did not work for all of my maps. I decided to take some ideas from it and write my own code. The result is in a [github repo](https://github.com/ana0209/map-fitting-2D). Here I explain the approach I took and why. I also analyze its shortcomings and talk about future work.


## Approach

In order to calculate a rigid transformation from one image to another we need 3 pairs of matching points. In this work I assume that a floorplan cannot be reflected which reduces the number of point pairs we need to two. This assumption is made because the reflected floorplan is not the same floorplan. For instance, if we have an apartment that has a bathroom to the right of the front door upon entering it and another apartment with the bathroom to the left to the front door upon entrance, we cannot say that these two apartments have the same layout. Floorplans of the same space can be matched by using only scaling, rotation and translation. 

Therefore, need only 2 pairs of corresponding points in two floorplans in order to estimate the transformation between them. 

The steps of my approach are summarized in the following pseudocode. It should be noted that ground_truth refers to the ground truth floorplan of the apartment whereas floorplan refers to the floorplan found by the SLAM algorithms.

```
distance_map = get_distance_map(ground_truth_image)

ground_truth_corners = calculate_corners(ground_truth_image)
floorplan_corners = calculate_corners(floorplan_image)

boundary_points = get_boundary_points(floorplan_image)
 
current_fitness_score = Inf
final_transform = NULL

foreach pair1 in pairs_of(gt_corners):
    foreach pair2 in pairs_of(floorplan_corners):
        if pair_can_be_skipped(pair1, pair2): 
		    # See Optimizing the number of comparisons for details
		    continue

        T = calculate_transform(pair1, pair2)
        transformed_boundary_points = transform(boundary_points, T)

        fitness_score = calculate_fitness_score(distance_map, \\ 
            transformed_boundary_points)

        if fitness_score < current_fitness_score:
            final_transform = T
            fitness_score = current_fitness_score

return final_transform
```

First, the corners are calculated for both images. In my case, as I had created the ground truth myself, manually, I actually loaded corners from a json, rather than calculating them from the ground truth image that was generated from the said corners. After that, each pair of corners in the ground truth is matched to each pair of corners in the floorplan, with some correspondences being discarded quickly by using heuristics to reduce running time. Then, using the two pairs, a transformation is calculated that transforms the floorplan image to the ground truth plan. Next, fitness score is calculated to evaluate the transform. Finally, the transform with the lowest fitness score is selected as the correct one.

The above pseudocode skips some implementation details and optimizations. They can be found in the subsequent sections.


### Finding the corners

I tried using [Harris corner detector in OpenCV](https://docs.opencv.org/3.4/dd/d1a/group__imgproc__feature.html#gac1fc3598018010880e370e2f709b4345), however it was very dependent on the parameters used and it would not work across different maps with the same set of parameters. 

So I decided to find corners indirectly, by first detecting lines and then checking for their intersections. I tried using [HoughLines](https://docs.opencv.org/3.4/dd/d1a/group__imgproc__feature.html#ga46b4e588934f6c8dfd509cc6e0e4545a) to detect the lines but had similar issue as with Harris corner detector. However, I found that by using [HoughLinesP](https://docs.opencv.org/3.4/dd/d1a/group__imgproc__feature.html#ga8618180a5948286384e3b7ca02f6feeb) function that detects line segments instead of lines more consistent results can be achieved.

Once the line segments are detected a check is run on every pair of them to see if they have vertices close to each other and if the angle between them is between 45 and 135 degrees. The assumption here is that angles smaller than that are likely to belong to line segments that were supposed to be a part of a single line segment that was split into two during the line segments detection. In this project, I assume that the angles between the walls are 90 degrees, which makes the possible angles between segments around 0 (they are parallel to each other) or around 90 degrees (they are perpendicular to each other). It would possible to detect corners even when the walls are not at 90 degree angle. This is done to an extent by keeping the allowable angle range so wide. If it was known that there are walls with an even larger angle between them, then the angle range could be widened. However one should always leave a decent range to account for parallel line segments to be rejected.

It is worth noting that this method does not necessarily detect all the corners in the image due to imperfect results in the line segments detection step. However, for the map fitting to work, it is enough that one pair of corners that are at a reasonable distance from each other in the fitted map to be detected correctly alongside with their counterparts in the ground truth image. In my case, I had all the ground truth corners manually obtained, so the only condition was to get at least 2 corners in the generated flooplan. The condition for the corners to be at a reasonable distance comes from the fact that for a small distance a small error in terms of pixels will be amplified in the calculated transform. In other words a small number of pixels makes a big difference in the calculated length and even orientation of a tiny segment connecting two corners, whereas an error of a few pixels makes much less of an impact on those two properties of a longer segment.

Here is pseudocode for finding corners:

```
calculate_corners(floorplan_image):
    line_segments = calculate_linesegments(floorplan_image)
    corners = []
    for i in [0, size(line_segments) - 1]:
        l1 = line_segments[i]
        for j in [i+1, size(line_segments) - 1]:
            l2 = line_segments[j]

            l1_end1, l1_end2 = vertices(l1)
            l2_end1, l2_end2 = vertices(l2)

            if 45 < angle(l1, l2) < 135:
                if close(l1_end1, l2_end1):
                    corners.add((l1_end1 + l2_end1)/2)
                elif close(l1_end1, l2_end2): 
                    corners.add((l1_end1 + l2_end2)/2)
                elif close(l1_end2, l2_end1): 
                    corners.add((l1_end2 + l2_end1)/2)
                elif close(l1_end2, l2_end2):
                    corners.add((l1_end2 + l2_end2)/2)

    return corners
```

The reason the corner is not necessarily an endpoint of the two linesegments is to account for the cases where the two segments are really close to each other and satisfy the angle condition to make a corner but they do not share exactly the same endpoint due to imperfections in the map image or the line segment detection. 


### Optimizing the number of comparisons

If we assume that the number of corners in the plan is *n* then the code runs in *O(n^4)* time. This is because each image has *n^2* corner pairs that need to be fitted to *n^2* corner pairs in the other image. This is less than ideal running time and can cause the code to run slowly even for small maps. For this reason, I skip some of the fittings when I can assume that they will be incorrect by using heuristics.

First, corners that are too close to each other are not used to determine the transform because the transform determined that way would be imprecise. Next, a scale factor from the corner pair distances is calculated before moving on to the rest of the transform. If the area of the floorplan we are fitting adjusted by the scale factor is significantly smaller than the ground truth image, we discard the corner pairs correspondence. There is an implicit assumption here that the ground truth image is tightly fitted around the plan itself. In case there is no noisy pixels far outside of the ground truth plan, you can use [crop_floorplan](https://github.com/ana0209/map-fitting-2D/blob/master/source/crop_floorplan.py) function in the github repo. This will give you a tightly bound ground truth floorplan.


#### Same scale optimization

In some cases we know that the ground truth map and the floorplan to be fitted have the same scale. In general we should know the scale of the ground truth (because it is ground truth!). The map resolution of the SLAM calculated maps is also known. In that case, we can easily discard all the corner pairs correspondences where the corner distance in one image is very diffent from the distance between the corners in the other image. This additionally reduces the runtime significantly.

This could be expanded to take in a known scale (even if it is not 1) and to make a similar check, discarding corner pair correspondences where the distances' ratio is not close to the input scale. This is left for future work. One way to go around this with the code as is is to resize the ground truth plan to the scale of the floorplan or vice versa before running the fitting algorithm and then using the same scale option in the code.


### Finding the transform

Transform is found piecewise. First, scaling factor is determined by dividing the distance between the corners in one image with the distance of the corners in the evaluated pair in the other image.
Next, rotation matrix is calculated using an OpenCV function [cv2.getRotationMatrix2D](https://docs.opencv.org/3.4/da/d54/group__imgproc__transform.html#gafbbc470ce83812914a70abfb604f4326). Translation is calculated by subtracting the coordinates of one of the corners in the first image from its scaled and rotated counter part. Rotation and scaling are applied together in a single matrix, while the translation is applied using subtraction on the scaled and rotated points. This is done because it was faster to apply translation this way than to combine it with scaling and rotation in a single matrix and to use that single matrix for the transform.


### Fitness score

Once transformation is calculated, a fitness score is obtained between the ground truth and the transformed map. Here is the pseudocode for calculating the fitness score:

```
distance_map = get_distance_map(ground_truth_image)

calculate_fitness_score(distance_map, transformed_boundary_points):
    fitness_score = sum(distance_map[transformed_boundary_points[0,:], 
        transformed_boundary_points[1,:]])
    fitness_score /= 
        size(transformed_boundary_points[0,:])

    return fitness_score
```

Ground truth map distance map is calculated at the beginning. It contains distance from every point to the closest boundary point in the ground truth map (example images). This way the fitness score calculation is sped up since all the possible distances are known before hand. Once the floorplan is transformed, one simply picks the distances from the map where the transformed boundary points of the floorplan fall and adds them up to get the fitness score before averaging the score out.

The image below shows the ground truth plan on the left, its distance map in the middle and inverted distance map to the right. I have added the inverted distance map because it is more visually appealing and easier to analyze than the very dark image of the distance map itself. When studying it, you should note that the whiter areas correspond to the smaller values i.e. smaller distances from the floorplan boundary whereas darker areas correspond to bigger distances from the floorplan boundary. 

![dist_map_figure](https://raw.githubusercontent.com/ana0209/ana0209.github.io/master/images/dist_map_figure.png)


## Examples of results

Here are some examples of fitting results:

![figure-good-floorplan](https://user-images.githubusercontent.com/51337969/67909587-f5164e00-fb3c-11e9-98d0-ea96096d26d3.png)

The images above showed fitting examples for correct maps. Here is an example of a poor map that was still fitted reasonably by the code:

![figure-bad-floorplan](https://user-images.githubusercontent.com/51337969/67909582-f0ea3080-fb3c-11e9-8206-59d0176ac8c5.png)


## Conclusions and future work

This code has been tested only on a small set of maps. One of the major future work points is having it tested on a large set of maps.

As part of a more extensive testing, running time limitations should be tested. And in light of that, different ways to optimize the could should be added. Thoughts that come to mind in how to do that are in terms of breaking the maps into smaller sections, doing the fitting for those sections and combining transforms for separate sections into a single transform with some low fitness score. Another thought is to not have the brute force comparison of all the corner pairs, but to do some sort of corner pair matching, much like is done in feature matching in computer vision or specifically in the area of map alignment in the [work of Shahbandi](https://github.com/saeedghsh/Map-Alignment-2D). 

Another simple extension was mentioned in the [Same scale optimization](#same-scale-optimization) section, that of having it take a known scale to speed up a fitting process even when that scale is not 1.
