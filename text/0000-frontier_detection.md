# Frontier detection for Cartographer

## Summary
[summary]: #summary

Introduce the paper on frontier detection and a possible plug-in system for Cartographer

## Motivation and approach
[motivation]: #motivation

This RFC is a bit different than usual: I want to share a paper I wrote about a special frontier detection algorithm I devised for Cartographer.
The pre-print of the paper is available on arXiv: https://arxiv.org/abs/1902.11061.
A monorepo with an implementation (research-quality :-)) is available at https://github.com/larics/cartographer_frontier_detection, and a demo video with Deutsches museum at https://goo.gl/62zEUy.

The paper describes everything in detail, but a TLDR of some cool details is that we perform naive (but vectorized with Eigen!) local edge detection in submaps when Cartographer updates them and test the resulting local frontier against the nearby intersecting submaps (whose bounding boxes are stored in an R-tree so we can quickly query for them).
When Cartographer performs optimization and moves stuff around, we efficiently reproject and retest the previously detected local frontier without having to perform local submap edge detection again.

## Integration into Cartographer
[approach]: #approach

The provided implementation is kind of a proof-of-concept which I do not expect to be merged as-is into Cartographer. 
However, it got me thinking how we could do it.

I could have written a separate node which would perform submap queries just like the RViz submaps plugin or the occupancy grid node.
However, the `Submap::ToResponseProto` RPC serialization and transfer of submaps is a real performance killer, and I don't think we could have achieved an average of 70-80 Hz frontier updates by going that way (and performance was a selling point of the method described in the paper).
By integrating frontier detection into the Cartographer node executable we can directly access the submaps in its memory.
And this means that this code has to live in a fork of cartographer_ros.

ROS has a solution for this in the form of nodelets running in a single node so they can efficiently share memory.
Another solution could be some kind of a plug-in system for Cartographer (currently the Cartographer node just calls the frontier detection code from a single place, in `OnLocalSlamResult`).
If we could define a plug-in interface, the frontier detection code could live as a standalone plugin, while still being able to efficiently access the submaps memory in the Cartographer node.
Perhaps the occupancy grid node could be another client which would benefit from an increase in performance if it was able to access the submap memory directly.

## Discussion

Any feedback on the paper is welcome.

Does it make sense to add a plug-in interface?
