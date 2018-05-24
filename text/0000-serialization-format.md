# Serialization Format

## Summary

[summary]: #summary

Definition of a versioned, backwards-compatible and extensible serialization
format for the state of Cartographer.

This RFC is a small deviation of the previously proposed header format from
[RFC-0015](https://github.com/googlecartographer/rfcs/blob/master/text/0015-serialization-header.md).

## Motivation

[motivation]: #motivation

At the moment, the file format for serializing the state of Cartographer is
defined purely within the serialization & deserialization routines of
Cartographer. Since we serialize all information into a binary stream of
`protobuf` messages (*pbstream*) without a format-version information, extending
or changing the format inevitably leads to a breaking change for Cartographer
users, as there is no way of falling back to load previously serialized states.

This RFC is going to address this issue by introducing a `SerializationHeader`
message and an interface for parsing serialized states, independent of the
file-format.

## Approach

[approach]: #approach

For being able to identify the underlying file-format, we introduce a
`SerializationHeader` message.

```
message SerializationHeader {
  string format_version = 1;
}
```

We further change the current `SerializedData` protobuf message, which is meant
to be able to contain `oneof` any of the message types required for serializing
the mapping state:

```
message SerializedData {
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

For backwards compatibility, we will rename the original `SerializedData`
definition to `LegacySerializedData`.

By keeping all the payload related data wrapped in a single message type, we can
also easily switch to a different underlying storage format in the future (e.g.
[Riegeli](https://github.com/google/riegeli/)).

### Generic File Format

Using the above messages, the generic file format is defined as:

*Generic File Format* |
:-------------------: |
SerializationHeader   |
(SerializedData)*     |

### Version 1.0 file-format

Using the definitions above, the proposed file-format for version 1.0 is as
follows

*Version 1.0 fileformat (proto order)* |
:------------------------------------: |
SerializationHeader                    |
PoseGraph                              |
AllTrajectoryBuilderOptions            |
(Submap)\*                             |
(Node)\*                               |
(ImuData)\*                            |
(OdometryData)\*                       |
(FixedFramePoseData)\*                 |
(TrajectoryData)\*                     |
(LandmarkData)\*                       |

### Backwards-Compatibility

*   To gracefully transition to the new format, we will provide an internal
    fallback to the *old* parsing of PbStreams. Meaning, we will first try to
    read the a `SerializationHeader` from the pbstream and if this fails, seek
    back in the stream and try parsing the *old* format. This way we can at
    least provide functionality for parsing the last pre-header version.
*   To discourage usage of the old format, the code for serializing the mapping
    state will only support the new format.
*   Additionally we will provide a migration tool to re-write already serialized
    pbstreams into the new format.

## Discussion Points

[discussion]: #discussion

*   Version string for new format: 0.9 or 1.0?
*   Duplicating protos to separate storage and *wire* format as of
    protobuf-best-practices
*   Naming preferences:
    -   `SerializationHeader`
    -   `SerializedData`
