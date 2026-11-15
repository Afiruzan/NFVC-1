# NFVC-1 Benchmark and Evaluation Protocol

**Status:** Proposed protocol. No benchmark results are included yet.

## 1. Purpose
Measure whether neural residual enhancement provides a reproducible rate-distortion benefit over the exact same base codec, then compare the complete system fairly against established codecs. Separate compression quality from runtime, memory, energy, and deployment complexity.

## 2. Comparison systems
At minimum evaluate:
- base codec alone;
- base plus neural enhancement;
- base plus enhancement and learned entropy coding;
- VVC reference software (VTM);
- AV1 reference encoder (AOM);
- AV2 reference encoder (AVM), when a stable build and fair configuration are available.

If NFVC uses AV1 as its base, do not attribute differences against a differently configured AV1 result to a new base codec. Report base-only and enhancement contributions separately.

Record source URL, release/version, exact commit, build flags, configuration files, command line, and patches for every codec.

## 3. Datasets and split policy
Candidate datasets include UVG, HEVC common test sequences (classes B/C/D/E), BVI-DVC, MCL-JCV, JVET material, and additional screen-content, text, face, animation, HDR, and out-of-distribution content where licensing permits.

Publish exact sequence manifests and dataset terms. Split by source sequence; never distribute adjacent frames from the same sequence across training and test. Keep test data untouched during tuning.

## 4. Operating points and bitrate
Use multiple operating points and select a meaningful common quality interval. Count actual emitted bytes:

\[
R_{total}=\frac{8(B_{base}+B_{enh}+B_{syntax})}{T}.
\]

Include all headers, model signaling, padding, and coder termination bytes. BD-rate is valid only over a common measured quality range; do not extrapolate to manufacture overlap. Report per-sequence results and describe aggregation.

## 5. Quality metrics
Report PSNR-Y/U/V, SSIM, MS-SSIM, VMAF with exact model/version, and optionally LPIPS and validated temporal flicker/consistency measures. Specify color conversion, chroma handling, bit depth, clipping, alignment, and metric implementation. Report SDR and HDR separately. Include controlled visual inspection; no single metric fully captures perceptual quality.

## 6. Complexity and deployment
On identical hardware/software conditions measure encode/decode time, per-frame latency distribution, throughput, jitter, dropped frames, peak resident memory, CPU/GPU/NPU utilization, energy where instrumented, model/binary size, initialization time, and thermal throttling.

Record hardware, OS, compiler, driver, power mode, thread count, batch size, warm-up, repetitions, and measurement method.

## 7. Required ablations
| ID | Configuration | Question |
|---|---|---|
| A0 | Base only | Reference |
| A1 | Base + neural residual, simple entropy | Does enhancement help? |
| A2 | A1 + learned entropy | Does it reduce actual bytes? |
| A3 | A2 + temporal latent prediction | Future experiment |
| A4 | A2 + ROI allocation | Future experiment |
| A5 | Wavelet branch | Optional independent experiment |
| A6 | Fractal branch | Optional independent experiment |
| A7 | Combined system | Only after individual contributions are understood |

Use the same test sequences and comparable operating points. Report negative results.

## 8. Robustness
Test truncated headers/payloads, invalid/excessive lengths, unknown types/profiles, checksum corruption, missing/duplicate enhancement units, reordering, scene cuts, recovery points, model hash mismatch, extreme dimensions, and malformed latent shapes. Base decoding must not depend on neural runtime or model files.

## 9. Reproducibility package
Archive source commit and diff status, exact commands, configs, model hashes, dataset manifest/preprocessing, raw per-sequence metrics, logs, hardware/software manifest, plotting scripts, failed runs, and exclusion reasons.

## 10. Claim discipline
The proposed 15–25% BD-rate improvement over VVC is an aspirational target only. It must not be described as achieved until supported by valid, reproducible experiments. Distinguish measured results from estimates, simulations, training losses, and hypotheses.
