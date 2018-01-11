# Cartographer AMCL replacement

## Summary
[summary]: #summary

Make cartographer in Pure localization mode works as a replacement for AMCL and be one of the ROS localization method.

## Motivation
[motivation]: #motivation

Cartographer is really powerful in term of localization and a pure localization mode was developed (it works already great). In order to be adopted by the ROS community it should be simple to run and works similarly  in its implementation as AMCL


## Approach
[approach]: #approach

* Use parameters to initialize the starting pose of the robot, similarly as ` ~initial_pose_x`, ` ~initial_pose_y`, `~initial_pose_a` in AMCL: http://wiki.ros.org/amcl
* Support the `/initialpose` topic to relocate the robot. This topic is for instance used out of the box by rviz `2D pose estimate` button (or keyboard shortcut: p). A quick and dirty implementation was done here: https://github.com/googlecartographer/cartographer_ros/pull/637
* Publish the static map against which localization is done on the `/map` topic for the navigation stack to be able to be run on top of it (global planner for instance) and to have a static reference to record waypoint in. Currently with the occupancy_grid node, the map which is published is a superposition of the static map + the current submap. A workaround is to not run the occupancy_grid node and in parallel run a `map_server` node which publish a *.pgm/*.png/ + *.yaml map that had to be generated from the same *.pbstream file which used by the pure localization node. It will be nice to have everything work in one step.
* Provide a way to easily edit a map afterwards (if this is possible). AMCL maps being bitmaps, there are easily cleaned manually, in GIMP for instance. This is very useful when maping a site which has moveable furnitures (chairs, people standing, christmas tree, etc...), we want to be able to remove these non fix objects from the localization map.
* Make the origin of the map changeable. It is currently the starting point of the trajectory that has been used for generating the localization map.

## Discussion Points
[discussion]: #discussion

