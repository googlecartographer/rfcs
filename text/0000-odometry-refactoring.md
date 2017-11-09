# Refactor odometry in SPG optimization problem

## Summary
[summary]: #summary

Do not use `std::vector<TransformInterpolationBuffer>` in SPG optimization problem anymore.

## Motivation
[motivation]: #motivation

`TransformInterpolationBuffer` does not support trimming-in-middle.

Performing refactoring of odometry data is a pre-requisite for serialization/deserialization.

## Approach
[approach]: #approach

Replace `std::vector<TransformInterpolationBuffer>` by `MapByTime<sensor::OdometryData>`?


## Discussion Points
[discussion]: #discussion

How to implement the interpolation functionality?

Expand `MapByTime`, or subclass it (`InterpolatedMapByTime`)?

Where to use `transform::StampedTransform`, and where to use `sensor::OdometryData`?
