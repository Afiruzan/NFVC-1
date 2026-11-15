# NFVC-1 Technical Specification — Draft v0.1

**Status:** Non-normative design proposal. **Implementation:** Not yet implemented. **Date:** 2026-11-15.

## 1. Abstract
NFVC-1 is a proposed layered hybrid video-coding system. A conventional block-based base codec generates an independently decodable reconstruction. An optional neural enhancement layer represents the error between the source and the actual decoded base reconstruction. The neural representation is quantized and entropy-coded. This document defines a research baseline, not an interoperable standard. No compression-performance claim is made.

## 2. Goals and non-goals
Goals: independent base decoding; enhancement relative to the decoded base output; complete rate accounting; deterministic encoder/decoder agreement; reproducible ablations.

Out of scope for v0.1: temporal latent prediction, fractal coding, wavelet branches, large Transformers, normative network transport, hardware acceleration, and a finalized public bitstream standard. Fractal coding may be studied as an isolated experiment, not assumed to be part of the core.

## 3. Input assumptions
The initial target is progressive YCbCr 4:2:0 video. 8/10-bit, HD/UHD, and common frame rates are intended evaluation targets, not currently verified capabilities. Signal or constrain dimensions, bit depth, chroma format, frame time base, color primaries, transfer characteristics, matrix coefficients, and range. Do not silently guess missing color metadata in conformance tests.

## 4. Layered coding equations
For source frame X_t, the base encoder emits B_t. A conforming base decoder reconstructs:

\[
\hat X_t^{(B)} = D_B(B_t;\phi_B)
\]

The enhancement residual must be computed against this actual decoded reconstruction:

\[
E_t = X_t - \hat X_t^{(B)}.
\]

The neural analysis transform produces:

\[
Z_t=g_a(E_t;\theta_a),\qquad \hat Z_t=Q(Z_t;\Delta).
\]

The synthesis transform reconstructs:

\[
\hat E_t=g_s(\hat Z_t;\theta_s),\qquad
\hat X_t=\operatorname{Clip}(\hat X_t^{(B)}+\hat E_t).
\]

Encoder and decoder must agree on arithmetic, color conversion, rounding, clipping, tensor layout, padding, and cropping. Do not form the residual from an ideal prediction or a different encoder-local reconstruction.

## 5. Base layer
AV1 is the initial candidate backend for rapid prototyping. If selected, the base payload is AV1 and base decoding is delegated to a compatible AV1 decoder. NFVC-1 in this configuration is a wrapper plus optional enhancement layer, not a new independently implemented base codec. A native base codec is future work and must be evaluated separately.

## 6. Neural enhancement
Use a compact CNN analysis/synthesis baseline. A possible starting point is 3×3 convolutions, channel widths 32/64/96, three downsampling stages, and latent spatial scale 1/8. These are tunable starting values, not proven optimal parameters.

Every model package must specify architecture/version, weight hash, input/output domain, tensor layout, padding, latent shape, quantizer, precision, and deterministic inference requirements. Begin with FP32 training; evaluate FP16/INT8 separately for quality, speed, memory, and energy.

## 7. Quantization and entropy coding
Uniform scalar latent quantization is a possible baseline. Quantization step sizes and rounding must be signaled or fixed by profile. The entropy model must be causal or otherwise reproducible from decoder-available context:

\[
p(\hat Z)=\prod_i p(\hat Z_i\mid\hat Z_{<i},c_i).
\]

A range or arithmetic coder may be used. Integer CDF construction, precision, symbol order, escape handling, termination, and byte alignment must be fixed before interoperability is claimed.

## 8. Rate-distortion objective
A research objective may be:

\[
\mathcal L=R_B+\alpha R_E+\lambda_1D_{pixel}
+\lambda_2D_{MS\text{-}SSIM}
+\lambda_3D_{perceptual}
+\lambda_4D_{temporal}.
\]

Start with actual coded rate and pixel/MS-SSIM distortion. Introduce perceptual and temporal terms through ablations. VMAF is primarily an evaluation metric unless a separately documented differentiable proxy is used. Count base, enhancement, syntax, model signaling, padding, and termination bytes.

## 9. Decoder independence and robustness
Base decoding must work without neural weights/runtime. Missing, corrupt, or unsupported enhancement should be skipped safely while preserving valid base output. Bounds-check lengths, dimensions, tile counts, model IDs, allocation sizes, and integer arithmetic. Define recovery behavior for temporal state before adding temporal coding.

## 10. Proposed software interfaces
A future C++ API may expose initialize, encode_frame, flush, reset, get_statistics, decode_packet, and release operations. Keep a base-only decoder separable from the full decoder. Python/PyTorch is proposed for training; C++ is the intended production-oriented decoder path. Neither is implemented yet.

## 11. Training and data separation
Split by source sequence, not randomly sampled adjacent frames. Keep test sequences untouched during tuning. Record dataset version/license, sequence list, preprocessing, crop policy, and split manifest. Progress from image residual experiments to residuals generated from actual decoded base output, then joint rate-distortion training.

## 12. Acceptance gates
Do not call this a working codec until it demonstrates base encode/decode and reload; independent base-only decoding; deterministic enhancement encode/decode; actual rate including overhead; measurable enhancement benefit at matched operating points; reproducible quality/runtime/memory and energy where available; and malformed/truncated stream tests.

## 13. Open decisions
Base backend, final CNN, latent dimensions, entropy coder, exact syntax, rate control, and license remain provisional. Record rationale and evidence for each decision.
