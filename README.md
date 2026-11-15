# NFVC-1 — Neural-Fractal Video Codec

**Status:** Research specification v0.1. This repository currently contains design documentation, not a working encoder or decoder.

NFVC-1 is a proposed layered hybrid video codec. A conventional block-based base codec produces an independently decodable reconstruction. An optional neural enhancement layer encodes the error between the source and the actual decoded base reconstruction.

**Research integrity:** No BD-rate, quality, speed, energy, or complexity result is claimed. The proposed 15–25% BD-rate improvement over VVC is an aspirational target, not a measured result or guarantee.

## Design principles
1. Base-layer decoding must not require neural weights or a neural runtime.
2. All headers, model signaling, padding, and entropy-coder termination count toward bitrate.
3. Record codec versions, commits, configurations, datasets, hardware, and metric versions.
4. Evaluate features through controlled ablations.
5. A missing or corrupt enhancement payload must not prevent valid base decoding.

## Proposed v0.1 scope
- Existing block-based codec backend, initially AV1 as a rapid-prototyping candidate.
- Compact CNN residual analysis/synthesis.
- Latent quantization and learned entropy coding.
- Experimental layered bitstream/container.
- Offline reproducible evaluation.

If AV1 is used, the base payload is AV1. In that configuration NFVC-1 is a wrapper/enhancement system, not an independently implemented base codec.

## Documentation
- [Technical specification](docs/specification_v0.1.md)
- [Architecture](docs/architecture.md)
- [Experimental bitstream syntax](docs/bitstream_syntax.md)
- [Benchmark protocol](docs/benchmark_protocol.md)

## Implementation status
| Component | Status |
|---|---|
| Architecture and scope | Documented proposal |
| Base codec integration | Not implemented |
| Neural model | Not implemented |
| Quantization and entropy coding | Not implemented |
| Bitstream writer/reader | Not implemented |
| C++ decoder | Not implemented |
| Conformance tests and benchmarks | Not available |

## License
No license has been selected or applied. Apache-2.0 may be considered after checking third-party codec, model, dataset, and patent obligations. Until a LICENSE file is added, do not assume these contents are licensed under Apache-2.0.
