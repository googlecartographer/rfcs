# Serialization Format

## Summary

[summary]: #summary

Definition of a stable, yet extensible serialization format for cartographer-maps.

*   pick-up proposed header format from [RFC-0015](https://gitub.com/googlecartographer/rfc/text/0015-serialization-header.md)

## Motivation

[motivation]: #motivation

Currently there is no way of identifying the format of the proto-stream file, given the file alone.

*   Getting rid of magic positions in the stream
*   Versioning of the serialized, s.th. we can support loading maps from
    previous versions (backwards compatibility)
*   Avoid breaking users in the future due to changes in storage format

## Approach

[approach]: #approach

* define Header proto including a version number
* Wrap all other protos using a one-of {...} proto.

* Given the version number, the order of protos in the pbstream file is uniquely defined and we can make hard assumptions about it
* When we want to extend the content, we only need to "bump" the version number and add the new protos to the one-of section

* Backwards-Compatibility:
	* If no header found, fallback trial to "pre-header" format
	* migration tool
		* should in theory be a trivial header "prepend"
* Code:
	* decouple PbStream reader from format parsing
		* Introduce new interface that allows format agnostic map-building and use this everywhere we read pbstreams for the purpose of posegraph reading
		
* New serialization / pose_graph writing will default to the new proposed format

* Moving away from PbStream to another (potentially more efficient) format on disk, should be treated in an independent RFC.

## Discussion Points

[discussion]: #discussion

* Separating storage from client format completely
	* Protobuf best practices advice to "completely" separate storage from wire protos from early on. We should therefore "duplicate" our serialization types as a follow up and only use them for storage and never for anything else.
* Naming
* Versioning: string? 0.9?

