# Serialization Format

## Summary

[summary]: #summary

Definition of a stable, yet extensible serialization format for maps.

*   pick-up proposed header format from
    [RFC-0015](https://gitub.com/googlecartographer/rfc/text/0015-serialization-header.md)

## Motivation

[motivation]: #motivation

*   Getting rid of magic positions in the stream
*   Versioning of the serialized, s.th. we can support loading maps from
    previous versions (backwards compatibility)
*   Avoid breaking users in the future due to changes in storage format

## Approach

[approach]: #approach

## Discussion Points

[discussion]: #discussion

*   Separating storage from client format completely
