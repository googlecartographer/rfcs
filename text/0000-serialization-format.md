# Serialization Format

## Summary

[summary]: #summary

Definition of a versioned, backwards-compatible and extensible serialization format for cartographer-maps.

This RFC is a small deviation of the previously proposed header format from [RFC-0015](https://gitub.com/googlecartographer/rfc/text/0015-serialization-header.md).

## Motivation

[motivation]: #motivation

At the moment, the file format for serialized *PoseGraphs* is defined purely within the serialization & deserialization routines of cartographer. Since we serialize all information into a binary stream of `protobuf` messages (*pbstream*) without a format-version information, extending or changing the format inevitably leads to a breaking change for cartographer users, as there is no way of falling back to load previously serialized maps.

This RFC is going to address this issue by introducing a `SerializationHeader` message and introducing an interface for parsing serialized *PoseGraphs*.

## Approach

[approach]: #approach

For being able to identify the underlying file-format, we introduce a `SerializationHeader` message.

```
message SerializationHeader {
  string format_version = 1;
}
```

We further introduce a `SerializedPoseGraphData` protobuf message, which is meant to be able to contain `one-of` any of the message types required for serializing the mapping state:

```
message SerializedPoseGraphData {
  oneof data {
    PoseGraph pose_graph = 1;
    AllTrajectoryBuilderOptions all_trajectory_builder_options = 2;
    Submap submap = 3;
    Node node = 4;
    ImuData imu_data = 5;
    OdometryData odometry_data = 6;
    FixedFramePoseData fixed_frame_pose_data = 7;
    TrajectoryData trajectory_data = 8;
    LandmarkData landmark_data = 9;
  }
}
```

By keeping all the `PoseGraph` related payload-data wrapped in a single message type, we can also easily switch to a different file format in the future (e.g. [Riegeli](https://github.com/google/riegeli/)).

### Generic File Format
Using the above messages, the generic file format is defined as:

| *Generic File Format* |
| :---: |
| SerializationHeader |
| (SerializedPoseGraphData)* |

### Version 1.0 file-format

Using the definitions above, the proposed file-format of version 1.0 is as follows

|*Version 1.0 fileformat (proto order)*|
| :---: |
| SerializationHeader | 
|PoseGraph |
| AllTrajectoryBuilderOptions |
| (Submap)\* |
| (Node)\* | 
| (ImuData)\*| 
| (OdometryData)\* |
| (FixedFramePoseData)\* |
| (TrajectoryData)\* |
| (LandmarkData)\* |


### Backwards-Compatibility

* To gracefully transition to the new format, we will provide an internal fallback to the *old* parsing of PbStreams. 
Meaning, we will first try to read the a `SerializationHeader` from the pbstream and if this fails, seek back in the stream and try parsing the *old* format. This way we can at least provide functionality for parsing the last pre-header version.
* To discourage usage of the old format, the code for serializing the mapping state will only support the new format.
* Additionally we will provide a migration tool to re-write already serialized pbstreams into the new format.

## Discussion Points

[discussion]: #discussion
* Version string for new format: 0.9 or 1.0?
* Duplicating protos to separate storage and *wire* format as of protobuf-best-practices
* Naming preferences:
   	- `SerializationHeader`
   	- `SerializedPoseGraphData`




