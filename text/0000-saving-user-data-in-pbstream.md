# Save user data in proto stream

## Summary
[summary]: #summary

Teach libcartographer to store additional user data from clients in the proto stream

## Motivation
[motivation]: #motivation

This RFC is a continuation of 0015 (#19), which introduced a header for proto streams that is able to store additional metadata such as the proto stream version.
The further step proposed in this RFC is to enable libcartographer clients to provide some additional data to be stored in the proto stream, which they can get back when the SLAM state is restored in a new Cartographer instance.
A motivating example is given in discussion of [cartographer_ros#1157](https://github.com/googlecartographer/cartographer_ros/pull/1157).

For example of cartographer_ros as a client of this functionality, we could embed the used ROS node and map builder options in the proto stream. 
When running cartographer_ros nodes with the "load saved state" option, the user would be able to tell cartographer_ros to use the node/map builder options embedded in the proto stream, thus freeing them from the necessity of keeping around the .lua tuning file corresponding to the proto stream.

## Approach
[approach]: #approach

Introduce a new proto, `UserData`, in `serialization.proto`, consisting of two fields: `type`, a type identifier (integer or string?), and `data`, a string containing the payload.
Clients of libcartographer are responsible for serializing their data into the `data` string.

`MapBuilder::SerializeState` will take an (optional) vector of `UserData` protos and insert them after the constant part of the proto stream (after the header, pose graph and all trajectory builder options), to keep compatibility with the current proto stream format.

`MapBuilder::LoadState` will return, besides the trajectory remapping like it currently does, the same vector of `UserData` protos that was passed during `SerializeState`.

cartographer_ros will introduce a new proto or ROS message (up for discussion) called `NodeOptions`, which corresponds to `cartographer_ros::NodeOptions` (which currently does not have serialization).
When saving the state, it will serialize its `NodeOptions` into a `UserData` proto and pass it to `MapBuilder::SerializeState`.
A new flag, `load_configuration_from_state` will be introduced, which can be used with `load_state_filename` instead of providing `configuration_directory` and `configuration_basename(s)`.

Also introduce `cartographer::io::DeserializeUserDataFromFile` analogously to `DeserializePoseGraphFromFile` to avoid a catch-22 - to construct a `MapBuilder`, we need the configuration; but to read the configuration, we need a `MapBuilder` to perform `MapBuilder::LoadState`.
`DeserializeUserDataFromFile` will then allow us to read the embedded configuration directly from the `.pbstream` before constructing the `MapBuilder`.


## Discussion Points
[discussion]: #discussion

Is the proposed API adequate?

Use a ROS message or a proto for serializing `NodeOptions`?
