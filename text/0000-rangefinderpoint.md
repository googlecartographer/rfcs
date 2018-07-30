# Introduce RangefinderPoint and RangeFinderPointWithTime

## Summary
[summary]: #summary

Introduce `RangefinderPoint`/`RangefinderPointwithTime` to be used throughout Cartographer instead of using `Eigen::Vector3/4` directly.

## Motivation
[motivation]: #motivation

Currently, Cartographer is using `Eigen::Vector3/4` directly in lots of places.
For example, a rangefinder point cloud is an `std::vector` of `Eigen::Vector3/4`.
Suppose one wants to extend rangefinder points passing through Cartographer to include some metadata (e.g. laser ID).
Currently, that's not possible, but if Cartographer consistently used a structure named e.g. `RangefinderPoint` for rangefinder points instead of `Eigen::Vector3`, it would be trivial for the user to extend Cartographer to do this. 

## Approach
[approach]: #approach

Introduce classes `RangefinderPoint` and `RangeFinderPointWithTime`. Replace all uses of `Vector3/4` with the new classes.

Implement `RangeFinderPointWithTime::DiscardTime` method, returning `RangefinderPoint`, which shall be used instead of calling `Vector4::head<3>`. 

Define `operator*(Rigid3, RangefinderPoint)` for e.g. tracking <-> local frame transformations.

Libcartographer users could then easily modify these functions to preserve their metadata.

## Discussion Points
[discussion]: #discussion

Inheritance of `Vector3/4` instead of composition?
Class naming?
