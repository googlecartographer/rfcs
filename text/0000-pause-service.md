# Add a pause/continue service for cartographer_ros

## Summary
[summary]: #summary

Allow to pause/continue the SLAM system with a remote procedure call.

## Motivation
[motivation]: #motivation

This feature mainly addresses long-term operation of the SLAM system (several hours or even days).
In this scenario, situations can occur where you want to pause the current SLAM trajectory for a short period of time.
For example, to run a necessary sensor calibration procedure that could interfere the SLAM system during live operation.

Definition of a pause:
* ignore incoming sensor streams
* don't run local SLAM
  * don't publish new poses
  * or: re-publish latest pose if required
* backend threads (optimizations) that don't use fresh data can continue

*Side-note:*
>Of course, this assumes that the hardware system is not unintentionally moved during the pause ("kidnap").
>But in a concrete system, this can be ensured by the entity that's calling the pause/continue service.
>For example, a robot's behavior controller can block the motors during the pause.

## Approach
[approach]: #approach

One could add two new ROS services:
* `/pause_slam`
  * brings the node into a paused state
  * could have an option `bool republish_latest_pose`
* `/continue_slam` (or `/resume_slam`...)

Or (IMO better) everything in a single `/set_operation_state` service with equivalent `pause`/`running` options.

Since this is a local behavior (see robot control example above), I don't think equivalent gRPC calls are necessary.
We could/should however add an interface to retrieve the current operation state from the system (`/get_operation_state` in ROS and sth. for gRPC).

## Discussion Points
[discussion]: #discussion

* Any concerns that a pause might break any component?
