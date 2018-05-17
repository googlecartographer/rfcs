# Serialization Format

## Summary

[summary]: #summary

Definition of a stable, yet extensible serialization format for maps.

*   pick-up proposed header format from [RFC-0015](https://gitub.com/googlecartographer/rfc/text/0015-serialization-header.md)

## Motivation

[motivation]: #motivation

*   Getting rid of magic positions in the stream
*   Versioning of the serialized, s.th. we can support loading maps from
    previous versions (backwards compatibility)
*   Avoid breaking users in the future due to changes in storage format

## Approach

[approach]: #approach

* define Header proto including
	* versioning
	* fields proposed in [RFC-0015](https://gitub.com/googlecartographer/rfc/text/0015-serialization-header.md)
* Best-effort backwards compatibility by fallback to "latest" version format
	* try reading header 
		* on failure: load with current loading mechanism
		* on success: instantiate Deserializer for version defined in header
	* provide map_migration_tool to convert existing maps to new format
* New serialization will default to the new proposed format
* Moving away from our own proto-to-disk-serialization (pbstream) is a separate topic which will not be addressed here

## Discussion Points

[discussion]: #discussion

*   Separating storage from client format completely

