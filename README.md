# DTF-Net

Geometry–temporal adaptive fusion for underwater-to-air image restoration
under water-surface fluctuations.

[中文说明](DTF_NET_README_zh.md)

> **Partial public release — fusion core only.** This repository contains the
> actual fusion architecture, a cached-input inference interface and a synthetic
> example. It does not contain experimental data, trained weights, the training
> pipeline, prior generation, or baseline/ablation workflows. It is not a complete
> reproduction bundle for the manuscript.

## Overview

An underwater camera observes above-water targets through a fluctuating water
surface. Time-varying refraction introduces geometric distortion and local blur.
The full DTF-Net pipeline combines a geometric template from Diffeomorphic
Template with a temporal restoration from TMT Stage1. A deformation-magnitude
map M and an aligned temporal-variation map U condition the fusion.

```text
Same raw observation window (20 frames in the research pipeline)
  -> external geometric processing -> D: template, M/U: priors
  -> external TMT Stage1            -> T: restored target frame

                       PUBLIC FUSION CORE
  D -> geometric encoder --+-- prior-guided gates at 3 scales -- decoder
  T -> temporal encoder ---+               ^                       |
  M,U ------------------------------------+-----------------------+
                                                                  v
                      O = clamp((1-g)*D + g*T + R, 0, 1)
```

At scale l, G_l = sigmoid(Conv3x3([Fd_l, Ft_l, M_l, U_l])) and
Z_l = (1-G_l)*Fd_l + G_l*Ft_l. The decoder predicts an image gate g and
a bounded residual R = residual_scale * tanh(residual_head).
Feature gates are channel-dependent; the image gate has one channel.
U is an auxiliary inconsistency cue, not a calibrated uncertainty probability.

## Core configuration

| Component | Supplied setting |
| --- | --- |
| RGB encoders | Separate parameters, widths 32/64/128 |
| Feature resolutions | 1, 1/2, 1/4 (stride rounding for odd dimensions) |
| Encoder blocks | Two 3x3 convolutions, GroupNorm, GELU |
| Priors | M and U, one channel each |
| Fusion | Learned feature gates and learned image gate |
| Residual bound | 0.25 by default |
| Initial heads | Feature gates/image gate/residual head are zero initialized |

Zero initialization makes an untrained model output (D+T)/2. This is expected
initial behavior, not evidence of learned restoration performance.

## Repository structure

```text
dtf_net/
  model.py                       Original fusion implementation, unchanged
  api.py                         Validated cached-input API and strict loading
  infer.py                       PNG inference entry point
options/test/fusion.json          Fusion configuration
examples/synthetic_demo.py        Random-tensor structural demonstration
tests/test_core.py                Shape, initialization, prior and loading checks
datasets/                        Input contract only; no experimental images
experiments/pretrained_models/   Instructions only; no weights
results/                         Output directory instructions
NOTICE.md                        Upstream attribution and release provenance
CITATION.md                      Citation status
```

## Installation

Python 3.10 or newer is required. In an isolated environment, install a PyTorch
build suitable for your hardware, then install the package from the repository root:

```bash
python -m pip install -r requirements.txt
python -m pip install -e . --no-deps
```

The requirements describe this public interface, not a claim about the exact
environment of the original training runs. CPU execution is supported.

## Synthetic example (no private files needed)

```bash
python -m examples.synthetic_demo
python -m unittest discover -s tests -v
```

The example uses random inputs and untrained weights. It prints tensor shapes;
it does not reproduce a manuscript image or metric.

## Input preparation

Prepare four files for the same observation window and target frame:

| Argument | Meaning | Tensor convention |
| --- | --- | --- |
| `--template` | Geometric template D | RGB, [B,3,H,W] |
| `--temporal` | TMT Stage1 target output T | RGB, [B,3,H,W] |
| `--flow` | Encoded deformation magnitude M | Grayscale, [B,1,H,W] |
| `--uncertainty` | Encoded aligned temporal-variation cue U | Grayscale, [B,1,H,W] |

The PNG example reads 8-bit images and divides integer values by 255, following
the supplied dataset loader's RGB/L decoding convention. It requires all images
to have the same resolution; it rejects size mismatches instead of resizing.
This stricter check is specific to the public example.

M/U must already have the encoding used to train the checkpoint. This release
does not define or reproduce the upstream prior-generation normalization.
Do not independently stretch or visualize these maps with a colormap. Raw flow
vectors are not valid substitutes for the encoded magnitude image. No ground
truth image is used by this fusion interface.

The full pipeline applies TMT to raw frames. The public cached-input interface
instead receives its precomputed target output T. Internally an identity adapter
and a singleton time dimension reuse the unchanged fusion implementation;
this is not a claim that the full method restores from a single raw frame.

## Inference with your compatible checkpoint

No weights are included. Place your compatible checkpoint outside the tracked
files or in experiments/pretrained_models/, and run (one line, works in PowerShell):

```bash
python -m dtf_net.infer --config options/test/fusion.json --template datasets/example/template_rgb.png --temporal datasets/example/stage1_center.png --flow datasets/example/flow_magnitude.png --uncertainty datasets/example/uncertainty.png --checkpoint experiments/pretrained_models/your_model.pth --output results/example --device cpu
```

The paths above illustrate your own files; they are not bundled sample data.
Use `--device cuda` when a suitable GPU runtime is available.
The loader accepts an original `model` checkpoint, a `state_dict` checkpoint,
or a direct fusion state dictionary. Only `stage1.*` tensors are excluded.
All fusion weights load strictly; incompatible architectures fail explicitly.
Loading uses `weights_only=True` and does not silently fall back to unrestricted
pickle loading. Original checkpoints may lack residual_scale metadata; configure
the training value explicitly (0.25 is the supplied source default).
Use T from the matching trained temporal branch, especially after joint tuning.

Outputs are out_center.png and image_gate.png. There is no full benchmark,
training command, or claim of reproducing the manuscript's numerical results.

## Reproducibility scope

Included: the fusion network, numerical conventions, cached-input inference,
configuration, synthetic example and tests. Original mode switches remain in
model.py to preserve its parameter layout; the public API uses the full gated model.

Not included: raw images, reference images, sample manifests and splits,
trained checkpoints, TMT implementation, geometric solver/prior generation,
training scripts and losses, baseline runs, ablation orchestration,
evaluation scripts, server launchers, internal logs or manuscript figures.

## Citation and acknowledgements

Publication metadata will be added when confirmed; see CITATION.md. The upstream
Diffeomorphic Template and TMT work is acknowledged in NOTICE.md. VascHAT informed
the repository organization only; its code and dataset are not used here.

## License

The included code is distributed under Apache License 2.0; see LICENSE and NOTICE.md.
This license applies to the released files, not to the withheld data, weights,
or external implementations.
