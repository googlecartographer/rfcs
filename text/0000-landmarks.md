# Landmark observations
Add landmarks support to Cartographer.

## Summary
[summary]: #summary
Cartographer will be able to work with a new type of sensor data that contains ID and location of landmarks.

## Motivation
[motivation]: #motivation
Landmarks translation/orientation can improve SLAM quality. Landmarks rendered on the map can be useful for inspection. 

## Approach
[approach]: #approach
Landmarks are sparse features of geometry that we want to map. Unlike dense range data, landmarks are assumed to be sparse in comparison. In order to support using them, current SLAM approach has to be generalzed. 

### Sensor
Each landmark will have an ID and a position relative to the robot with or without orientation. A landmark observation can contain several landmarks at a time.

### Submap
Submaps for 2D and 3D geometries are occupancy grids that contain accumulated range data.  There can be two approaches to add landmarks to the submaps:

1. Submaps hold a dictionary with landmark data keyed by landmark IDs;
2. Every cell of the grid is modified to hold various features, for example, likelihood of landmarks, ML features. 

The second approach is more generic and makes it possible to introduce landmarks based on the strength of the signal, e.g. Wi-Fi signal, BT beacons. The first approach should work faster, it is also more flexible and non-intrusive.
  
### Constraint builder
`ConstraintBuilder` should use landmarks while adding intra-submap constraints:

* `FastCorrelativeScanMatcher` will use landmarks in the bounding function;
* `CeresScanMatcher` will have an additional term in its cost function.

### Optimization problem
Landmarks may enter sparse pose graph optimization as additional independent variables. It would increase the dimension of the optimization by `number_of_landmarks * number_of_parameters_per_landmark`, which could be undesirable when there is a significant number of landmarks.

There is another use case here. If Cartographer is provided with relative positions of landmarks with respect to each other, it can be added to the sparse pose graph optimization similarly to `FixedFramePoseData`.

### Rendering
Landmarks IDs and positions should be propagated to RViz for rendering.

## Discussion Points
[discussion]: #discussion
Should the landmarks be a part of local SLAM if a robot is capable to detect landmarks while moving?