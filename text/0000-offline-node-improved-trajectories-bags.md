# Improved handling of multiple trajectories and bags in the offline node

## Summary
[summary]: #summary

The offline node currently supports multiple sequential (non-concurrent) trajectories via loading multiple bags, where each bag is a separate trajectory.
This RFC aims to give the user greater flexibility when dealing with multiple robots, multiple (possibly concurrent) trajectories and multiple bags.

## Motivation
[motivation]: #motivation

Let's give an overview of all possibilities how a dataset layout and content can look like:

| Robot plurality                                                         | Trajectory plurality                                 | Bag plurality                                                          |
|-------------------------------------------------------------------------|------------------------------------------------------|------------------------------------------------------------------------|
| __A.__ Single robot                                                     | __1.__ Single trajectory                             | __a.__ Single bag                                                      |
| __B.__ Multiple identical robots  (using the same sensor topics/frames) | __2.__ Multiple trajectories, necessarily sequential | __b.__ Multiple continuous bags  (like a single bag) - a bag aggregate |
| __C.__ Multiple different robots  (different sensor topics/frames)      | __3.__ Multiple trajectories,  possibly concurrent   | __c.__ Multiple bags                                                   |

The online node currently supports the following combinations:
__A1__, __A/B 2__ and __C 2/3__ (using `StartTrajectory` and `FinalizeTrajectory` service calls)

The offline node currently supports: __A1a__, __A/B 2c__.

The online node can never support __B3__, because only single robot's data can be published on a topic at a time.
Also, __B3__ and __a/b__ is impossible for the same reasons.

Note that some combinations are invalid, such as __A3__ (a single robot can't be on multiple concurrent trajectories), and __B/C 1__ (there can't be a single trajectory if there are multiple robots).
Furthermore, __A/B 2 a/b__ would require us to be able to specify when a trajectory stops and begins within the same bag aggregate and on the same range data topics, which would be too complicated and out of scope.

The aim of the RFC is to enable the user to use the offline node for __C2/3__ like in the online node, to preserve the ability for __A/B 2c__, and to possibly offer additional functionality: for example, __b__, and __B3c__, which is otherwise impossible with the online node.

## Approach
[approach]: #approach

The proposed offline node configuration for multiple, possibly concurrent trajectories (__3__) should closely follow the way of the online node for achieving this, listed here for reference:
  - Performing multiple calls of the `StartTrajectory` service, with possibly different `TrajectoryOptions` and necessarily different sensor data topics (__C__); all is specified in the service request.
  - Alternatively, the `start_trajectory` helper node is invoked multiple times, with possibly different `.lua` files for `TrajectoryOptions` and necessarily different sensor data topics (__C__); the recently merged PR googlecartographer/cartographer_ros#584 enables specifying different topics via topic remapping.
  - Specially, if a trajectory is finalized, we may start another independent trajectory which uses the same sensor data topics and possibly different `TrajectoryOptions` (__B2__).
  This is already achievable in the offline node by simply loading multiple bags, which creates a trajectory for each bag, although `TrajectoryOptions` are necessarily the same for all trajectories (__B2c__). 
  The ability to do this has to be preserved or expanded.

If we suppose that there is a mandated 1:1 relationship between bags and trajectories in the offline node, the following configuration is proposed.

### Configuration #1

For __each bag (trajectory)__, the configuration is a tuple consisting of:
  - `TrajectoryOptions` (given in a `.lua` file)
  - A set of sensor data topics
  - Boolean `use_bag_transforms` which allows us to ignore transforms in the bag
  - An optional `.urdf` file.

Both `TrajectoryOptions` and the sensor data topics can be identical for multiple different bags (preserving __B2c__). However, in addition to that, the trajectories can be concurrent (__B3c__).

Because the trajectories may be concurrent, due care would need to be taken to ensure that data published on the same topic, but in different bags, gets routed to the appropriate trajectory.
Also, each trajectory should have its own transform buffer, to avoid interference between different bags.
  - For example, we may have two identical separate robots driving concurrently (__B3__); all the frames and topics are identical for both robots.
  The online node cannot handle this (we would have to ensure that the sensor data topics would be different by e.g. using different namespaces for each robot, as well as having different frames; conflicting transforms with same frames would be an unsolvable nightmare). 
  - However, if we had a separate bag for each robot (__B3c__), the offline node would have the advantage of knowing which data originated from which bag, so it could handle this case.

Going further, allowing multiple trajectories in a single bag (__C 2/3 a__) and relaxing the 1 trajectory : 1 bag restriction would mean that the configuration of each bag becomes a set of tuples mentioned above.
This is given in Configuration #2.

### Configuration #2

Similar to Configuration #1, but have a set of tuples for each bag.
Each tuple describes a trajectory in the bag.

For both Configuration #1 and Configuration #2, all concurrent trajectories (whether in the same bag as permitted by Configuration #2, or in different bags) are be __collated__.

### Handling split bags

Another common use case is (__b__) - a single continuous trajectory is split into multiple bags (e.g. when using the split option of `rosbag record`).

This could be handled by defining a `BagAggregate`: a list of one or more continuous bags, which are to be treated as one bag.

### Configuration #3
Configuration is given as a set of tuples for each `BagAggregate`, each tuple describing a trajectory in the `BagAggregate`.

As before, any concurrent trajectories are collated.
Each `BagAggregate` has its own transform buffer (which is shared by all bags in the aggregate).

### Data format of the configuration

The configuration can be given in a `.lua` file, similar to `assets_writer`.

## Discussion Points
[discussion]: #discussion

Should Configuration #1, #2 or #3 be used?
