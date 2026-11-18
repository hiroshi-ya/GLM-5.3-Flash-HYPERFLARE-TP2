---
license: other
license_name: nvidia-open-model-agreement
license_link: https://huggingface.co/nvidia/GLM-5.3-Flash-FP4-Local
pipeline_tag: image-text-to-text
tags:
- hyperflare
- flare
- glm-5.3-flash
- fp4
- nvfp4
- sm121
- gb10
- tensor-parallel
- long-context
- image-text-to-text
base_model: nvidia/GLM-5.3-Flash-FP4-Local
---

# GLM-5.3-Flash FP4 HYPERFLARE — 1M, TP2 (two DGX Sparks)

A prepared [Hyperflare](https://github.com/NVIDIA/hyperflare-inference) runtime
package for **GLM-5.3-Flash** on two **DGX Sparks** (GB10 / SM121), tensor-parallel TP2. The base checkpoint is
[`nvidia/GLM-5.3-Flash-FP4-Local`](https://huggingface.co/nvidia/GLM-5.3-Flash-FP4-Local)
at `76348ddb`. This package can be served with full **1,048,576-token** context.

## Before you spend time reading this

If you need a TLDR, you can find some [llama-benchy](https://github.com/eugr/llama-benchy) results generated on a dual Asus GX10 setup [here](BENCH.md).

If you are still interested after reading that, here are some additional reasons on why you may want to read through this:
1) You don't enjoy using docker or even virtual environments.
2) You don't enjoying messing with endless amount of environment variables.
3) vLLM comes with its own glitches and you are too tired to deal with those.
4) vLLM loads too slowly, especially during cold boots.
5) You want to play with Hyperflare on your onw, but its (AI-written) documentation is too confusing and lengthy to follow.

## How to serve

If you just want to download this package and serve it, here are the prerequisites:
1) A relative new pair of `rustc` and `cargo`.
2) Two DGX Sparks (FE or variants) connected via ConnectX7. For this package, tensor parallel cannot be set to 4, and if you have such a need, you need to build your own package (see [Reproduction](#reproduction)).
3) At least ~185 GiB of available storage space: `hyperflare` needs about 10 GiB to compile, and the package is about 180 GiB.
4) A way to download large files from huggingface.

> [!TIP]
> If you are low on storage space, once you obtained the `flare` executable binary, you can delete the source code and build artifacts. The binary itself is about 26 MiB. Besides, consider exporting your download directory as an NFS mount point, so you can not only save space but also avoid copying the model.

### Obtain `flare`
```console
$ git clone https://github.com/NVIDIA/hyperflare-inference
$ cd hyperflare-inference
$ # git checkout 5a4e8679 # in case a breaking change happen
$ cargo build --release --manifest-path flare-cli/Cargo.toml
```
After a success compilation, the binary will be generated `hyperflare-inference/flare-cli/target/release/`.

### Obtain this model package
If you have the `hf` CLI tool:
```console
$ hf download hiroshiya/GLM-5.3-Flash-HYPERFLARE-GB10-TP2 \
    --local-dir your/desired/path/
```
I'm not sure if `flare` 

# Reproduction

ss