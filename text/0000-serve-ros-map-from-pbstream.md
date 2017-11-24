# Serve ROS map from a pbstream file

## Summary
[summary]: #summary

We would like to have a ROS node that serves a `nav_msgs/OccupancyGrid` message like the `occupancy_grid_node` does.
But instead of publishing the current state of cartographer, a frozen state from a pbstream file should be read and compiled into a flat map, which is then published.


## Motivation
[motivation]: #motivation

Since `OccupancyGrid` is the standard message type for map, many nodes depend on such a topic.
If the pbstream gets frequently updated re-generating a `.pgm` and `.yaml` file in oder to read it in again using a ROS `map_server` is administrative overhead and generates redundant data. 

## Approach
[approach]: #approach

This feature could either be implemented by modifying `occupancy_grid_node` to accept an alternative input as a pbstream file,
or by modifying `pbstream_to_ros_map` to publish a rostopic if a flag is set. Conceptually the first option is less intrusive I would argue.
Either way this probably may require some refacturing of one or both those nodes code, so it could be reused.

## Discussion Points
[discussion]: #discussion

* Whether to modify `occupancy_grid_node` or `pbstream_to_ros_map` or even to introduce another binary.
