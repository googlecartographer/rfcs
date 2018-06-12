# Add a monitoring bridge and interface to cartographer_ros

## Summary
[summary]: #summary

Add a bridge from `cartographer_ros` to the monitoring facilities of `cartographer`.
Provide an ROS interface to retrieve metrics from `cartographer_ros`.

## Motivation
[motivation]: #motivation

Interfaces for collecting runtime metrics have been added to the Cartographer core library with the implementation of [RFC-0014](https://github.com/googlecartographer/rfcs/blob/master/text/0014-monitoring.md).
As of today, this interface can be used to expose metrics via Prometheus' protocol in the cloud map builder server.

This RFC aims to expose Cartographer's metrics locally on robot instances that run `cartographer_ros` using a suitable ROS interface.
We want to provide those metrics from the Cartographer node to other nodes in a suitable message format.

Use-cases include:
  
  * Analyzers that deduce status from the collected metrics and publish to ROS' standard `/diagnostics` channel ([REP-107](http://www.ros.org/reps/rep-0107.html)), e.g. for monitoring the localization quality on a robot.
  
  * Loggers that store the collected metrics in a database for further analysis.

## Approach
[approach]: #approach

We don't implement a `/diagnostics` publisher, as this would require implicit reasoning about the metrics.
Instead, we provide a service to query the metrics.

### Internal

* Add a registry in form of a `cartographer_ros::metrics::FamilyFactory` that implements the [abstract base class](https://github.com/googlecartographer/cartographer/blob/master/cartographer/metrics/family_factory.h) from the Cartographer library.

* Register it to the static global family factory stubs in the Cartographer library using `::cartographer::metrics::RegisterAllMetrics` at initialization of a node, i.e. before any computations take place.


### Public

* Add suitable message formats for the metrics to `cartographer_ros_msgs/msg/metrics` (to be defined, WIP).

* Add a service `/collect_metrics` that collects the recent metrics from the registry in its callback and replies with a metrics message.

## Discussion Points
[discussion]: #discussion

* 
