# NFVC-1 Experimental Bitstream Syntax — Draft

**Status:** Illustrative proposal only. Not normative, frozen, or interoperable.

## 1. Requirements
Base and enhancement payloads must be distinguishable. Base-only decoders must be able to skip enhancement units. Validate all lengths and identifiers before processing. Model/profile identity must be explicit. Unknown optional units should be skippable when framing is valid. Count all syntax overhead in bitrate.

## 2. Proposed unit envelope
| Field | Width | Meaning |
|---|---:|---|
| sync_code | 32 bits | Synchronization value, TBD |
| unit_type | 8 bits | Unit type |
| flags | 8 bits | Reserved flags |
| header_length | 16 bits | Header length in bytes |
| payload_length | 32 bits | Payload length in bytes |
| sequence_number | 32 bits | Sequence identifier |
| frame_id | 32 bits | Frame identifier |
| layer_id | 8 bits | Base/enhancement |
| reserved | 8 bits | Zero in this draft |
| checksum | 32 bits | Algorithm and coverage TBD |

Multi-byte integers are proposed as big-endian. Sync value, checksum, coverage, maximum unit size, alignment, and overflow policy must be fixed before implementation claims conformance.

## 3. Proposed unit types
| Value | Name | Purpose |
|---:|---|---|
| 0x01 | SEQUENCE_HEADER | Sequence properties |
| 0x02 | MODEL_CONFIG | Enhancement profile/model identity |
| 0x10 | FRAME_HEADER | Frame timing and flags |
| 0x20 | BASE_FRAME | Base codec frame |
| 0x21 | BASE_TILE | Reserved future base tile |
| 0x30 | ENHANCEMENT_FRAME | Neural enhancement payload |
| 0x31 | ENHANCEMENT_TILE | Reserved future enhancement tile |
| 0x40 | RECOVERY_POINT | Recovery metadata |
| 0x7F | END_SEQUENCE | Sequence termination |

Tile units are reserved and not required by the first prototype.

## 4. Sequence and model metadata
A sequence header should define syntax version/profile, dimensions, chroma format, bit depth, frame time base, color metadata, base codec identifier/version, enhancement presence, and decoder resource limits.

Model configuration should identify architecture/version, model-weight hash, latent shape/layout, quantization, entropy model/coder, integer CDF precision, symbol order, and inference format. Do not rely on a mutable model name or implicit download.

## 5. Frame association
Each enhancement unit must reference the same sequence and frame as its base frame. Define duplicate, missing, reordered, and late-unit behavior. The first offline prototype may require strict decode order.

## 6. Defensive parsing
Before allocation or decode:
1. Validate sync and fixed-header availability.
2. Validate header length and configured maximum.
3. Validate payload length against remaining bytes and maximum.
4. Check integer additions for overflow.
5. Validate sequence/frame/layer association.
6. Verify checksum.
7. Validate profile and decoded tensor dimensions.

A malformed enhancement unit should be discarded without preventing decoding of an intact base unit. A malformed base unit cannot be assumed recoverable from enhancement data.

## 7. Compatibility
No third-party implementation should be expected to parse this draft. Before freezing a public format, publish exact field semantics, state machines, checksum behavior, entropy coder details, conformance vectors, and versioning/extension policy.
