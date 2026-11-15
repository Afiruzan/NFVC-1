# NFVC-1 Architecture

**Status:** Proposed research architecture. No implementation or performance result is implied.

## 1. Overview
NFVC-1 has two independently meaningful layers:
1. Base path: a conventional block-based codec producing a decodable reconstruction.
2. Neural enhancement path: a learned representation of the difference between the source and the decoded base reconstruction.

## 2. Encoder flow
The base encoder produces B_t. The encoder locally decodes that bitstream using the same decoder behavior expected at the receiver:

\[
\hat X_t^{(B)}=D_B(B_t;\phi_B),\qquad E_t=X_t-\hat X_t^{(B)}.
\]

The enhancement path is:

\[
E_t\xrightarrow{g_a}Z_t\xrightarrow{Q}\hat Z_t
\xrightarrow{\text{entropy encode}}P_t^{(E)}.
\]

The mux combines base and enhancement units with sequence/frame identifiers and profile metadata.

## 3. Decoder flow
Demultiplex the stream and decode the base payload first:

\[
B_t\xrightarrow{D_B}\hat X_t^{(B)}.
\]

If the enhancement profile is supported and the payload is valid:

\[
P_t^{(E)}\xrightarrow{\text{entropy decode}}\hat Z_t
\xrightarrow{g_s}\hat E_t,
\quad \hat X_t=\operatorname{Clip}(\hat X_t^{(B)}+\hat E_t).
\]

Otherwise return the base reconstruction. Enhancement failure must not invalidate a valid base frame.

## 4. Logical modules
| Module | Responsibility | Status |
|---|---|---|
| Base encoder adapter | Invoke selected base codec | Planned |
| Base decoder adapter | Produce exact base reconstruction | Planned |
| Residual generator | Source minus decoded base output | Planned |
| Neural analysis transform | Map residual to latent | Planned |
| Latent quantizer | Quantize latent symbols | Planned |
| Entropy model/coder | Encode/decode symbols | Planned |
| Neural synthesis transform | Reconstruct residual | Planned |
| Bitstream mux/demux | Package and validate units | Planned |
| Evaluator | Rate, quality, complexity metrics | Planned |

## 5. Rate-distortion control
For each operating point, log base bytes, enhancement bytes, syntax/container overhead, total bytes and duration, codec settings, model/quantizer profile, quality metrics, encode/decode time, and peak memory. Estimated entropy-model rate is not actual bitrate; emitted bytes are authoritative.

## 6. Initial neural baseline
A compact CNN is preferred for the first implementation. Three downsampling stages and channel widths 32/64/96 are tentative starting values. Temporal latent prediction and ROI coding are deferred until the frame-independent baseline is measured. Fractal coding is an optional independent experiment.

## 7. Failure isolation
- Missing enhancement: output base reconstruction.
- Unknown profile: skip enhancement and report unsupported profile.
- Corrupt enhancement: reject enhancement unit; preserve base output.
- Corrupt base: follow base codec error behavior; do not invent a reconstructed frame.
- Model hash mismatch: reject enhancement unless explicit compatibility rules exist.

## 8. Proposed source organization
- codec/base, codec/neural, codec/bitstream, codec/encoder, codec/decoder
- models
- datasets (manifests/scripts only; do not commit restricted datasets)
- benchmarks, tests
- results/raw, results/metrics, results/reports

This is a proposed organization, not a report of existing source files.
