# Replace frozen trajectory from file

## Summary
[summary]: #summary

We propose to add `PoseGraphInterface::DeleteTrajectory` and `MapBuilderInterface::LoadStateFromFile` because these functions are missing to replace a frozen trajectory ("map") at run-time.

## Motivation
[motivation]: #motivation

Currently, it is not possible to replace a finished (and frozen) trajectory at run-time.
In fact, the trimmers are the only components that even delete parts of the pose graph.
When we add these two functions to the public API, the use case of swapping a frozen map at run-time is possible.
A caller can load a new frozen map, wait for global SLAM to process it, and delete the old frozen map.
Another use case is to clean up old trajectories on a device that runs continuously and sometimes starts and finishes trajectories.

## Approach
[approach]: #approach

In `PoseGraphInterface`, we add `virtual void DeleteTrajectory(int trajectory_id) = 0;`.
The deletion is dispatched until previous work items have finished.
Only finished trajectories can be deleted.

In `MapBuilderInterface`, we add `virtual void LoadFrozenStateFromFile(const std::string& filename) = 0;`.
Contrary to `LoadState`, the handler reads the file, so it can be called by stubs written in languages other than C++ and there is less gRPC traffic.

We also add all gRPC endpoints and a test that actually deletes a finished trajectory.


## Discussion Points
[discussion]: #discussion

