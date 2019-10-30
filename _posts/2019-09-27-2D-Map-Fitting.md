---
layout: post
title: "2D Map Fitting"
date: 2019-09-27
---

This project arose when I was trying to compare several 2D lidar based SLAM algorithms. You can find the code here https://github.com/ana0209/map-fitting-2D. I was having a robot move around a part of my home with a lidar mounted on top of it. I tested gmapping, Cartographer SLAM and Karto SLAM. I had generated the ground truth map manually. I wanted to fit the calculated maps by each of the algorithms to the ground truth map. It looked simple, but I could not find any ready made, accurate code out there. I found this project: https://github.com/saeedghsh/Map-Alignment-2D that looks interesting but it did not work for all of my maps. I decided to take some ideas from it and write my own code. This is the result.


# Approach

In order to calculate a rigid transformation from one image to another we need 3 pairs of matching points. In this work I assume that a floorplan cannot be reflected which reduces the number of point pairs we need to two. This assumption is made because I think that a reflected floorplan is not the same floorplan. For instance, if we have an apartment that has a bathroom to the right of the front door upon entering it and another apartment with the bathroom to the left to the front door upon entrance, we cannot say that these two apartments have the same layout. Floorplans of the same space can be matched by using scaling, rotation and translation only. 

We need only 2 pairs of points in two floorplans in order to calculate the transformation. 

The steps of my approach are summarized in the following pseudocode

```
distance_map = get_distance_map(ground_truth_image)

gt_corners = calculate_corners(ground_truth_image)
floorplan_corners = calculate_corners(floorplan_image)

boundary_points = get_boundary_points(floorplan_image)

current_fitness_score = Inf
final_transform = NULL

foreach pair1 in pairs_of(gt_corners):
	foreach pair2 in pairs_of(floorplan_corners):
		if pair_can_be_skipped(pair1, pair2): # See Optimizing the number of comparisons for details
			continue

		T = calculate_transform(pair1, pair2)
		transformed_boundary_points = transform(boundary_points, T)

		fitness_score = calculate_fitness_score(distance_map, transformed_boundary_points)

		if fitness_score < current_fitness_score:
			final_transform = T

return final_transform
```

First, the corners are calculated for both images (or in case of ground truth, they can be obtained from a manually created file). Then each pair of corners in the ground truth is matched to each pair of corners in the floorplan, with some correspondences being discarded quickly by using some heuristics. Then, using the two pairs, a transformation is calculated that transforms the floorplan image to the ground truth plan. Next, fitness score is calculated to evaluate the transform. Finally, the transform with the lowest fitness score is selected as the correct one.

The above pseudocode skips some implementation details and optimizations. They can be found in the subsequent sections.


## Finding corners

I tried using Harris corner detector in OpenCV (link), however it was very dependent on the parameters used and it would not work across different maps. 

So I decided to find corners indirectly, by first detecting lines and then checking for their intersections. I tried using HoughLines (link) to detect the lines but had similar issue as with Harris corner detector. However, I found that by using HoughLinesP function that detects line segments instead of lines that I get more consistent results.

Once the line segments are detected a check is run on every pair of them to see if they have vertices close to each other and if the angle between them is between 45 and 135 degrees. The assumption here is that angles smaller than that are likely to belong to line segments that were supposed to be a part of a single line segment that was split into two during the line segments detection. In this project, I assume that the angles between the walls are 90 degrees, which makes the possible corners between segments around 0 or around 90 degrees. It would be possible to detect corners even when the walls are not at 90 degree angle. This is done to an extent by keeping the allowable angle range so wide. If it was known that there are walls with an even larger angle between them, then the angle range could be widened. However one should always leave a decent range to account for parallel line segments to be rejected.

It is worth noting that this method does not necessarily detect all the corners in the image. However, for the map fitting to work, it is enough that one pair of corners that are at a reasonable distance from each other to be detected correctly in the map to be fitted. For ground truth, we have all the corners because they are manually added in our case. However, in case corners need to be detected in the ground truth map as well, it is important that the corresponding pair of corners is detected. Ground truth maps have better quality so this should be easy.

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


## Optimizing the number of comparisons

If we assume that the number of corners in the plan is *n* then the code runs in *O(n^4)* time. This is because each image has *n^2* corner pairs that need to be fitted to *n^2* corner pairs in the other image. This is less than ideal running time and can cause the code to run slowly even for small maps. For this reason, I skip some of the fittings when I can assume that they will be incorrect anyway by using heuristics.

First, corners that are too close to each other are not used to determine the transform because the transform determined that way would be imprecise. Next, we calculate the scale factor from the line segments lengths before calculating rest of the transform. If the area of the floorplan we are fitting adjust by the scale factor is significantly smaller than the ground truth image, we discard the corner pairs correspondence. There is an implicit assumption here that the ground truth image is tightly fitted around the plan itself. In case there is no noisy pixels far outside of the ground truth plan, you can use crop_floorplan function in the github repo (link). This will give you a tightly bound ground truth floorplan.


### Same scale optimization

In some cases we know that the ground truth map and the floorplan to be fitted have the same scale. In general we should know the scale of the ground truth and the map resolution of the SLAM calculated maps is also known. In that case, we can easily discard all the corner pairs correspondences where the corner distance in one image is very diffent from the distance between the corners in the other image. This additionally reduces the runtime significantly.

This could be expanded to take in a known scale (even if it is not 1) and to make a similar check, discarding corner pair correspondences where the distances' ratio is not close to the input scale. This is left for future work. One way to go around this with the code as is is to resize the ground truth plan to the scale of the floorplan or vice versa before running the fitting algorithm and then using the same scale option in the code.


## Fitness score

Once transformation is calculated, a fitness score is obtained between the ground truth and the transformed map. Here is the pseudocode for calculating the fitness score:

```
distance_map = get_distance_map(ground_truth_image)

calculate_fitness_score(distance_map, transformed_boundary_points):
	fitness_score = sum(distance_map[transformed_boundary_points[0,:], transformed_boundary_points[1,:]])
	fitness_score /= size(transformed_boundary_points[0,:])

	return fitness_score
```

Ground truth map distance map is calculated that contains distance to the closest boundary point in the ground truth map (example images). This way the fitness score calculation is sped up since all the possible distances are known before hand. Once the floorplan is transformed, one simply picks the distances from the map where the transformed boundary points of the floorplan fall and adds them up to get the fitness score before averaging the score out.

[[ground truth map and its distance map]]


# Examples of results

Here are some examples of fitting results:

[[example 1]]

The images above showed fitting examples for correct maps. Here is an example of a poor map that was still fitted in a reasonable fashion by the code:

[[example 2]]


# Conclusions and future work

This code has been tested only on a small set of maps. One of the major future work points is having it tested on a large set of maps.

As part of a more extensive testing, running time limitations should be tested. And in light of that, different ways to optimize the could should be added. Thoughts that come to mind in how to do that are in terms of breaking the maps into smaller sections, doing the fitting for those sections and combining transforms for separate sections into a single transform with some low fitness score. Another thought is to not have the brute force comparison of all the corner pairs, but to do some sort of corner pair matching, much like is done in feature matching in computer vision or specifically in the area of map alignment in the work of Shahbandi (https://github.com/saeedghsh/Map-Alignment-2D). 

Another simple extension was mentioned in Same scale optimization section (link), that of having it take a known scale to speed up a fitting process even when that scale is not 1.