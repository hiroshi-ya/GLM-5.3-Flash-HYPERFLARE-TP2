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
1. You don't enjoy using docker or even virtual environments.
2. You don't enjoying messing with endless amount of environment variables.
3. vLLM comes with its own glitches and you are too tired to deal with those.
4. vLLM loads too slowly, especially during cold boots.
5. You want to play with Hyperflare on your onw, but its (AI-written) documentation is too confusing and lengthy to follow.

## How to serve

If you just want to download this package and serve it, here are the prerequisites:
1. A relative new pair of `rustc` and `cargo`.
2. Two DGX Sparks (FE or variants) connected via ConnectX7. For this package, tensor parallel cannot be set to 4, and if you have such a need, you need to export your own package (see [Reproduction](#reproduction)).
3. At least ~185 GiB of available storage space: `hyperflare` needs about 10 GiB to compile, and the package is about 180 GiB.
4. A way to download large files from huggingface.
5. Working Nvidia driver (R580 or later) on both nodes.

> [!TIP]
> If you are low on storage space, once you obtained the `flare` executable binary, you can delete the source code and build artifacts. The binary itself is about 26 MiB. Besides, consider exporting your download directory as an NFS mount point, so you can not only save space but also avoid copying the model.

### Package properties

| Model                | GLM-5.3-Flash                    |
| -------------------- | -------------------------------- |
| license              | nvidia-open-model-agreement      |
| architecture         | sm_121                           |
| context              | 1,048,576 tokens                 |
| prefill chunk        | 4,096 tokens                     |
| concurrency          | 4 requests · 2 ranks             |
| default output limit | remaining context                |
| vocabulary           | 154,880                          |
| cache histories      | continuation state, paged latent |
| speculation          | mtp · widths 1, 2, 3             |

> [!IMPORTANT]
> You cannot freely modify the properties of this packaged runtime. For example, the `--mtp-draft-tokens` was set to 3 during the export, meaning that `flare` cannot use more than 3 speculative tokens when serving the model. Same applies to the concurrency and prefill chunk properties. If you want to adjust the properties, you have to export your own package. See [Reproduction](#reproduction).
> This package 

### Connect your DGX Sparks
If you have not done so, connect your DGX Sparks with a QSFP-56 cable to utilize the 200 GbE ConnectX-7. After you plugged in the cable, configure your local network connection. Refer to Nvidia's [guide](https://build.nvidia.com/spark/connect-multiple-sparks/overview) if you don't know how to.

> [!NOTE]
> CUDA runtime and NCCL are not required for serving. `flare`'s cross-node parallelism runs directly on RoCE.

### Obtain `flare`
```console
$ git clone https://github.com/NVIDIA/hyperflare-inference
$ cd hyperflare-inference
$ # git checkout 5a4e8679 # in case a breaking change happen
$ cargo build --release --manifest-path flare-cli/Cargo.toml
```
After a successful compilation, the `flare` executable binary will be generated in `hyperflare-inference/flare-cli/target/release/`.

### Obtain this model package
If you have the `hf` CLI tool:
```console
$ hf download hiroshiya/GLM-5.3-Flash-HYPERFLARE-GB10-TP2 \
    --local-dir /your/desired/path/
```
I'm not sure if `flare` automatically detects the default huggingface cache directory, hence the `--local-dir`.

If you do not have `hf`, `flare` itself comes with a `download` function. You can read on how to use it either by running `flare download -h` or referring to the [documentation](https://github.com/NVIDIA/hyperflare-inference/blob/main/docs/cli-design.md#downloading-published-cores).

### Package structure
```
GLM-5.3-Flash-HYPERFLARE-GB10-TP2
├── BENCH.md
├── glm-5.3-flash-hyperflare-gb10-tp2.safetensors
├── publisher.json
└── README.md
```
`glm-5.3-flash-hyperflare-gb10-tp2.safetensors` is the packaged hyperflare runtime, and `publisher.json` is the publisher key. When serving the model for the first time, you will have to trust the publisher first, or `flare` will refuse to serve the model.

### Serve
Start the head node first:
```console
$ export HEAD_IP=       # IP address of your head node
$ export HEAD_PORT=     # which port to use for head-worker communication
$ export SERVER_IP=     # IP address of your serving address
$ export SERVER_PORT=   # Port of your serving address
$ export MODEL_PATH=    # /path/to/GLM-5.3-Flash-HYPERFLARE-GB10-TP2
$ flare run $MODEL_PATH/glm-5.3-flash-hyperflare-gb10-tp2.safetensors \
    --publisher-keys $MODEL_PATH/publisher.json \
    --rendezvous $HEAD_IP:$HEAD_PORT \
    --host $SERVER_IP \
    --port $SERVER_PORT \
    --rank 0 \
    --gpus 0
```

Then start your worker node:
```console
$ export HEAD_IP=       # IP address of your head node
$ export HEAD_PORT=     # which port to use for head-worker communication
$ export MODEL_PATH=    # /path/to/GLM-5.3-Flash-HYPERFLARE-GB10-TP2
$ flare run $MODEL_PATH/glm-5.3-flash-hyperflare-gb10-tp2.safetensors \
    --publisher-keys $MODEL_PATH/publisher.json \
    --rendezvous $HEAD_IP:$HEAD_PORT \
    --rank 1 \
    --gpus 0
```

When serving for the first time, `flare` will ask you if you trust the publisher. Once you trust the publisher, `flare` will generate/update the `trust.toml` in `~/.config/flare/`, and the `--publisher-keys` argument will be no longer needed in the future.

## Reproduction

If you want to export the model by yourself, here are the prerequisites:
1. A relative new pair of `rustc` and `cargo`, and `uv`.
2. Two DGX Sparks (FE or variants) connected via ConnectX7.
3. At least ~350 GiB of available storage space: the original [`nvidia/GLM-5.3-Flash-FP4-Local`](https://huggingface.co/nvidia/GLM-5.3-Flash-FP4-Local) needs about 177 GiB, and the export needs about 171 GiB.
4. A way to download large files from huggingface.
5. Working Nvidia driver (R580 or later) on both nodes.

### Build tools
Install the following packages:
```console
# apt-get install build-essential cmake ninja-build pkg-config protobuf-compiler libssl-dev python3-dev liburing-dev
```
If on Fedora:
```console
# dnf install @development-tools cmake ninja-build pkgconf-pkg-config protobuf-compiler openssl-devel python3-devel liburing-devel
```

### Obatin `ignition` and `flare`
```console
$ git clone https://github.com/NVIDIA/hyperflare-inference
$ cd hyperflare-inference
$ # git checkout 5a4e8679 # in case a breaking change happen
$ uv sync --locked --extra prepare --extra test
$ cargo build --release --manifest-path flare-cli/Cargo.toml
```
