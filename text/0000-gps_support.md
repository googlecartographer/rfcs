# NavSatFix (GPS) Support in Cartographer ROS

## Summary
[summary]: #summary

The goal is to add NavSatFix (GPS) support to Cartographer ROS. 
The NavSatFix (GPS) data will be used in pose graph optimization, using the fixed frame pose feature of Cartographer.

## Motivation
[motivation]: #motivation

In outdoor applications, especially cars, GPS data is commonly available. 
Using this data will reduce the error in location, especially over large distances. 

## Approach
[approach]: #approach

We will wire up the [sensor_msgs/NavSatFix](http://docs.ros.org/api/sensor_msgs/html/msg/NavSatFix.html) and introduce two new options for this: `use_nav_sat` and `nav_sat_sampling_ratio`.

These messages will be translated into a common cartesian coordinate frame (e.g. ECEF) and given to Cartographer as `FixedFramePoseData`.

## Discussion Points
[discussion]: #discussion

