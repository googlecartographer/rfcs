# Landmark observations
Add landmarks support to Cartographer.

## Summary
[summary]: #summary
Cartographer will be able to work with a new type of sensor data that contains IDs and locations of landmarks.

## Motivation
[motivation]: #motivation
Landmarks translation/orientation can improve SLAM quality. Landmark visualizations can be useful for inspection.

## Approach
[approach]: #approach
Landmarks are sparse features of geometry that we want to map. Unlike dense range data, landmarks are assumed to be sparse in comparison. In order to support using them, current SLAM approach has to be generalized. 

### Sensor
Each landmark will have an ID and a position relative to the robot with or without orientation. A landmark observation can contain several landmarks at a time.

### Optimization problem
Landmarks may enter sparse pose graph optimization as additional independent variables. It would increase the dimension of the optimization by `number_of_landmarks * number_of_parameters_per_landmark`, which could be undesirable when there is a significant number of landmarks.

There is another use case here. If Cartographer is provided with the relative positions of landmarks with respect to each other, it can be added to the sparse pose graph optimization similarly to `FixedFramePoseData`.

### Rendering
Landmarks IDs and positions should be propagated to RViz for rendering.

## Discussion Points
[discussion]: #discussion
Should the landmarks be a part of local SLAM if a robot is capable to detect landmarks while moving?