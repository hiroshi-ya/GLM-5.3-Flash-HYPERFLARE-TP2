---
license: other
license_name: nvidia-open-model-agreement
license_link: https://www.nvidia.com/en-us/agreements/enterprise-software/nvidia-open-model-agreement/
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

# GLM-5.3-Flash FP4 HYPERFLARE — TP2, GB10

A prepared [Hyperflare](https://github.com/NVIDIA/hyperflare-inference) (at `1a53275`) runtime
package for **GLM-5.3-Flash** running across two **DGX Sparks** (GB10 / SM121), and a detailed guide on how to serve/reproduce it. The base checkpoint for the model is
[`nvidia/GLM-5.3-Flash-FP4-Local`](https://huggingface.co/nvidia/GLM-5.3-Flash-FP4-Local)
at `76348ddb`. This package can be served with vision support, a full **1,048,576-token** context, and around **4.5 million KV tokens**.

[![Static Badge](https://img.shields.io/badge/github-repo-black?style=for-the-badge&logo=github)](https://github.com/hiroshi-ya/GLM-5.3-Flash-HYPERFLARE-TP2) &nbsp; [![Static Badge](https://img.shields.io/badge/hugging%20face-model%20card-%23FFD21E?style=for-the-badge&logo=huggingface)](https://huggingface.co/hiroshiya/GLM-5.3-Flash-HYPERFLARE-GB10-TP2)

## Before you spend time reading this

If you need a TLDR, you can find some [`llama-benchy`](https://github.com/eugr/llama-benchy) results generated on a dual Asus GX10 setup in [BENCH.md](BENCH.md).

If you are still interested after browsing that, here are some additional reasons why you might want to read through this:
1. You don't enjoy using Docker.
2. You don't enjoy messing with an endless amount of environment variables.
3. vLLM comes with its own glitches, and you are too tired to deal with them.
4. vLLM loads too slowly, especially during cold boots.
5. You want to play with Hyperflare on your own, but its (AI-written) documentation is too confusing and lengthy to follow.

## How to serve

If you just want to download this package and serve it, here are the prerequisites:
1. A relatively new pair of `rustc` and `cargo`.
2. Two DGX Sparks (FE or variants) connected via ConnectX7. For this package, tensor parallel cannot be set to 4; if you need TP=4, you must export your own package (see [Reproduction](#reproduction)).
3. At least ~185 GiB of available storage space: `hyperflare` needs about 10 GiB to compile, and this package is about 181 GiB.
4. A way to download large files from huggingface.
5. Working Nvidia driver (R580 or later) on both nodes.

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
| speculation          | mtp · widths 1, 2, 3, 4, 5, 6    |

> [!IMPORTANT]
> You cannot freely modify the properties of this packaged runtime. For example, the `--mtp-draft-tokens` was set to 5 during the export, meaning that `flare` cannot use more than 5 speculative tokens when serving this package. The same applies to the concurrency and prefill chunk properties. If you want to adjust the model properties, you have to export your own package. See [Reproduction](#reproduction).

### Connect your DGX Sparks
If you have not done so, connect your DGX Sparks with a QSFP-56 cable to utilize the 200 GbE ConnectX-7. After you plug in the cable, properly configure your RoCE setup. Refer to Nvidia's [guide](https://build.nvidia.com/spark/connect-multiple-sparks/overview) if you don't know how to.

> [!NOTE]
> CUDA runtime and NCCL are not required for serving. `flare`'s cross-node parallelism runs directly on RoCE.

### Obtain `flare`
Install the following build dependencies on both nodes:
```console
# apt install build-essential cmake ninja-build pkg-config protobuf-compiler libssl-dev python3-dev liburing-dev libibverbs-dev
```
If on Fedora:
```console
# dnf install @development-tools cmake ninja-build pkgconf-pkg-config protobuf-compiler openssl-devel python3-devel liburing-devel rdma-core-devel libnl3
```

Then, on one node:
```console
$ git clone https://github.com/NVIDIA/hyperflare-inference
$ cd hyperflare-inference
$ # git checkout 1a53275 # in case a breaking change happens
$ cargo build --release --manifest-path flare-cli/Cargo.toml
```
After a successful compilation, the `flare` executable binary will be generated in `hyperflare-inference/flare-cli/target/release/`. You can either compile from source again on your second node, or simply copy the executable binary over there if both nodes are running the same Linux distribution.

> [!TIP]
> If you are low on storage space, once you have obtained the `flare` executable binary, you can delete the source code and build artifacts (saves ~9.5 GiB). The binary itself is about 26 MiB. Besides, consider exporting your download directory as an NFS mount point, so you can not only save space but also avoid copying the model manually.

### Obtain this model package
If you have the `hf` CLI tool:
```console
$ hf download hiroshiya/GLM-5.3-Flash-HYPERFLARE-GB10-TP2 \
    --local-dir /your/desired/path/
```
I'm not sure if `flare` automatically detects the default huggingface cache directory, hence the `--local-dir`.

If you do not have `hf`, `flare` itself comes with a `download` function. You can read on how to use it either by running `flare download -h` or referring to the [documentation](https://github.com/NVIDIA/hyperflare-inference/blob/main/docs/cli-design.md#downloading-published-cores).

>[!IMPORTANT]
>`flare` can only serve a core that's properly exported via `ignition`, meaning that you cannot use `flare` to directly serve the original `nvidia/GLM-5.3-Flash-FP4-Local` checkpoints. That said, the original checkpoints are not needed to serve this hyperflare package.

### Package structure
```
GLM-5.3-Flash-HYPERFLARE-GB10-TP2
├── BENCH.md
├── glm-5.3-flash-hyperflare-gb10-tp2.safetensors
├── publisher.json
└── README.md
```
`glm-5.3-flash-hyperflare-gb10-tp2.safetensors` is the packaged hyperflare runtime (yes, a single file), and `publisher.json` is the publisher key. When serving the model for the first time, you will have to trust the publisher first, or `flare` will refuse to serve the model.

### Serve
Start the head node:
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

Start the worker node:
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
If you do not specify the model path on the worker node, `flare` will copy the necessary files from the head node to the worker node for you automatically.

When serving for the first time, `flare` will ask you if you trust the publisher (on both nodes). After you trust the publisher, `flare` will generate/update the `trust.toml` in `~/.config/flare/` by default, and the `--publisher-keys` argument will no longer be needed in the future.

After the server is ready:
```console
$ curl http://$SERVER_IP:$SERVER_PORT/v1/models # check if alive
$ curl http://$SERVER_IP:$SERVER_PORT/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "nvidia/GLM-5.3-Flash-FP4-Local",
    "messages": [
      {"role": "user", "content": "hi how are you"}
    ],
    "temperature": 1.0,
    "top_p": 0.95
  }'
```

## Reproduction

If you want to export the hyperflare core by yourself, here are the prerequisites:
1. A relatively new pair of `rustc` and `cargo`, and `uv`.
2. Two or four DGX Sparks (FE or variants) connected via ConnectX7.
3. At least ~365 GiB of available storage space: the original [`nvidia/GLM-5.3-Flash-FP4-Local`](https://huggingface.co/nvidia/GLM-5.3-Flash-FP4-Local) needs about 177 GiB, and the export needs about 181 GiB.
4. A way to download large files from huggingface.
5. Working Nvidia driver (R580 or later) on all nodes.

> [!WARNING]
> TP=3 is likely not supported.

### Build dependencies
Install the following build dependencies:
```console
# apt install build-essential cmake ninja-build pkg-config protobuf-compiler libssl-dev python3-dev liburing-dev libibverbs-dev
```
If on Fedora:
```console
# dnf install @development-tools cmake ninja-build pkgconf-pkg-config protobuf-compiler openssl-devel python3-devel liburing-devel rdma-core-devel libnl3
```

### Reproduction flow
1. Obtain the original model checkpoints.
2. Use `ignition` to export the original checkpoints into a hyperflare core.
3. Sign the exported core (optional).
4. Convert the core into a packed form (optional).
5. Use `flare` to serve the hyperflare core.

### Obtain `ignition` and `flare`
On all nodes:
```console
$ git clone https://github.com/NVIDIA/hyperflare-inference
$ cd hyperflare-inference
$ # git checkout 1a53275 # in case a breaking change happens
$ uv sync --locked --extra prepare --extra test
$ cargo build --release --manifest-path flare-cli/Cargo.toml
```
> [!NOTE]
> `uv sync` will pull the CUDA toolkit and NCCL to the virtual environment, but the project will prioritize the host-native CUDA. `ignition` may work without a system-wide `nvcc`, but that has not been verified here.

### Export the core
On all nodes:
```console
$ pwd # make sure you are at the root of hyperflare source
/path/to/hyperflare-inference

$ . .venv/bin/activate # activate the virtual environment
```

Next, you will need to create an `agent.token` file in `~/.config/flare` on all nodes. According to the [documentation](https://github.com/NVIDIA/hyperflare-inference/blob/main/docs/multi-host.md#prepare-the-core):
> The file needs identical contents of at least 32 random bytes and mode `600` on each host.

Apparently, the `agent.token` check is designed to make sure that networking is secure during ignition. I'm not sure what the best practice is, so use your own creative way.

Before we export the core, there is another important step. `ignition` needs a huge amount of VRAM to perform the export, and because DGX Spark uses shared memory for CPU and GPU, CUDA may throw `CUDA_ERROR_OUT_OF_MEMORY` even if there is available memory. Therefore, it's recommended that you terminate all unnecessary programs and flush the page cache on all nodes before performing the export:
```console
$ sync && echo 3 | sudo tee /proc/sys/vm/drop_caches
```

After that, we are ready to export the core. Start the head node first:
```console
$ export HEAD_IP=   # IP address of your head node
$ export HEAD_PORT= # which port to use for head-worker communication
$ ignition export nvidia/GLM-5.3-Flash-FP4-Local \
    --rendezvous $HEAD_IP:$HEAD_PORT \
    --max-concurrency 4 \
    --mtp-draft-tokens 5 \
    --ranks 2 \
    --gpus 0
```
For a TP=4 (quad DGX Sparks) setup, pass `--ranks 4` instead of `--ranks 2`.

Then, start your (first) worker node:
```console
$ export HEAD_IP=   # IP address of your head node
$ export HEAD_PORT= # which port to use for head-worker communication
$ ignition join $HEAD_IP:$HEAD_PORT \
    --ranks 2 \
    --rank 1
```
For a TP=4 (quad DGX Sparks) setup, pass `--ranks 4` instead of `--ranks 2`, and adjust `--rank <N>` accordingly.

>[!NOTE]
> 1. `ignition` will check the default huggingface download cache directory. If `ignition` isn't able to locate the checkpoints locally, it will download the checkpoints for you. If your checkpoints are saved in a customized location, replace `nvidia/GLM-5.3-Flash-FP4-Local` with `/path/to/the/model`.
> 2. Hyperflare (as of `1a53275`) can only export from one specific version of GLM-5.3-Flash: [`nvidia/GLM-5.3-Flash-FP4-Local`](https://huggingface.co/nvidia/GLM-5.3-Flash-FP4-Local). That said, you cannot use checkpoints like `nvidia/GLM-5.3-Flash-NVFP4` and `RedHatAI/GLM-5.3-Flash-NVFP4`.
> 3. The export procedure can take a long time (about an hour for TP=2 setup).


If you want finer control, refer to the full argument list:
```console
$ ignition export glm-5.3-flash -h
```

After a successful export, inspect the model by running:
```console
$ flare models list
$ flare inspect nvidia/GLM-5.3-Flash-FP4-Local 
```

### Sign the core
This step is optional if you don't plan to publicly distribute your exported core.

First, create an encrypted production signing key:
```console
$ ignition keygen \
    --name 'Publisher Name'\
    --namespace 'publisher-hugging-face-namespace'\
    --directory /path/to/your/key
```

After that, you will obtain `signing-key.pem` and `publisher.json` in the directory you specified. You need `signing-key.pem` (and its passphrase) to sign the core, and the public needs `publisher.json` to serve the core that's signed by you.

Before signing the core, we need to find out where the exported core is:
```console
$ flare models list

Local models
  Storage                   /home/username/.cache/flare/cores
  Available                 1 model · 1 version

nvidia/GLM-5.3-Flash-FP4-Local
  Version                   gb10-tp2-202611190439  [DEFAULT]

[rest of the output trimmed]
```
If you haven't created a customized `config.toml`, `~/.cache/flare/core` will be the default location for exported cores, and the core for each specific model will be exported to `~/.cache/flare/core/MODEL/VERSION`.

Then, proceed to sign the core:
```console
$ ignition sign /path/to/your/flare/cores/models/nvidia_2FGLM-5.3-Flash-FP4-Local/gb10-tp2-202611190439 \
  --output /path/to/your/signed/glm-5.3-flash \
  --signing-key /path/to/your/signing-key.pem \
  --namespace your-name \
  --official-source https://huggingface.co/your-name/your-repo
```
Here, `nvidia_2FGLM-5.3-Flash-FP4-Local` is the name of the `MODEL` (`/` is replaced by `_2F`), and `gb10-tp2-202611190439` is `MODEL`'s `VERSION`. Replace `nvidia_2FGLM-5.3-Flash-FP4-Local/gb10-tp2-202611190439` with the actual `MODEL/VERSION`.

>[!TIP]
>`ignition sign` uses hard links for the model weights, so signing the core does not take much storage spaces.

After signing the core, inspect its properties:
```console
$ flare inspect /path/to/your/signed/glm-5.3-flash --all | less
```
In the `CORE` section, `publisher` should be `Your Name (your-namespace)`, and the signing key should be a `published key`.

### Pack the core
This step is optional. However, if you are using shared storage for storing models (via NFS, NVMe-oF, etc.), I strongly recommend you pack your core into a single `.safetensors` file.

```console
$ ignition convert /path/to/your/signed/glm-5.3-flash \
    --output /path/to/your/output.safetensors \
    --form packed
```

If you didn't sign your core and don't know where your exported unsigned core is, go back to [Sign the core](#sign-the-core) to find out how to locate your unsigned core.

After the conversion, the generated `.safetensors` file will include everything needed for `flare` to serve.

>[!WARNING]
>Packing the core requires about 181 GiB of additional storage space. Leave enough space for the conversion.

### Serve

Before serving across all nodes, you need to make sure that the publisher identification `json` file is available on all nodes.

If you did not sign your core:
- During the `ignition export` process, `ignition` will generate the development key **on the head node only**. You need to manually copy the `development-publisher.json` to your worker node(s). The default location is `~/.config/flare`. After copying the file to all nodes, you do not need to specify `--publisher-keys` to `flare run`.

If you signed your core:
- Make sure to copy the `publisher.json` to your worker node(s).

Start the head node first:
```console
$ export HEAD_IP=       # IP address of your head node
$ export HEAD_PORT=     # which port to use for head-worker communication
$ export SERVER_IP=     # IP address of your serving address
$ export SERVER_PORT=   # Port of your serving address
$ flare run /path/to/your/output.safetensors \
    --publisher-keys /path/to/your/publisher.json \
    --rendezvous $HEAD_IP:$HEAD_PORT \
    --host $SERVER_IP \
    --port $SERVER_PORT \
    --rank 0 \
    --gpus 0
```
If you didn't pack your core, replace `/path/to/your/output.safetensors` with `/path/to/your/signed/glm-5.3-flash` or `nvidia/GLM-5.3-Flash-FP4-Local@VERSION`. `VERSION` can be obtained from `flare models list`, and if there is only one version, you can drop the `@VERSION`.

Then, start your (first) worker node:
```console
$ export HEAD_IP=       # IP address of your head node
$ export HEAD_PORT=     # which port to use for head-worker communication
$ flare run /path/to/your/output.safetensors \
    --publisher-keys /path/to/your/publisher.json \
    --rendezvous $HEAD_IP:$HEAD_PORT \
    --rank 1 \
    --gpus 0
```
If you have two more worker nodes, adjust `--rank <N>` accordingly.

If you are not using a shared storage for holding the `output.safetensors`, you can drop `/path/to/your/output.safetensors` and let `flare` do the head-to-worker copy for you. If you didn't pack your core, you **have to drop the model path** and let `flare` copy the necessary files for your worker node:
```console
$ flare run \
    --publisher-keys /path/to/your/publisher.json \
    --rendezvous $HEAD_IP:$HEAD_PORT \
    --rank 1 \
    --gpus 0
```

When serving for the first time, `flare` will ask you if you trust the publisher. After you trust the publisher, `flare` will generate/update the `trust.toml` in `~/.config/flare/` by default, and the `--publisher-keys` argument will no longer be needed in the future.

### Publish
If you want to publish your exported core, don't forget to include the `publisher.json` file in your repository.

### Caveats
For a TP=2 setup, after `ignition export`, both the head and the worker node will have their export cache stored at `~/.cache/flare` by default. The head node will cache the whole model, while the worker node will cache roughly half the model (`rank-1` part). This is a good design, but it seems that the worker node `flare` does not know how to utilize `ignition` cache. No matter how I tried to execute `flare run` on my worker node, `flare` would always pull the `rank-1` part from the `rank-0` node, despite the existence of `ignition` cache.

Besides, when the exported core is not in packed form but in directory form, even if the entire package is stored at the same location on both nodes, `flare run /path/to/model` on worker node will still pull the `rank-1` part from the `rank-0` node. Even worse, if one specifies the model path, it seems that the worker node `flare` will assign a `MODEL@VERSION` to the pulled cache based on the finish time of head-to-worker copy, meaning that there will be a "version mismatch" and `flare` will refuse to serve. Hence, it's mentioned in the [Reproduction -> Serve](#serve-1) that one should drop the model path completely and avoid copying the package manually if the core is not packed.

TP=4 export and serving behavior is not verified, but will likely behave the same.

## Credits

This repository is a packaging, validation, and publication effort only. The model, runtime, recipes, kernels, and benchmark tooling are the work of others. No authorship claim is made over any upstream work.

- **Local Inference Lab and the NVIDIA Hyperflare (formerly Flare) authors and contributors** — runtime, recipes, package format, and documentation ([`NVIDIA/hyperflare-inference`](https://github.com/NVIDIA/hyperflare-inference)).
- **NVIDIA and Z.ai** — the GLM-5.3-Flash FP4 checkpoint ([`nvidia/GLM-5.3-Flash-FP4-Local`](https://huggingface.co/nvidia/GLM-5.3-Flash-FP4-Local/tree/main)) and the [`zai-org/GLM-5.3-Flash`](https://huggingface.co/zai-org/GLM-5.3-Flash) base model.
- **Ricky Brent** — [`rickybrent/GLM-5.3-Flash-HYPERFLARE-TP2`](https://huggingface.co/rickybrent/GLM-5.3-Flash-HYPERFLARE-TP2) inspired this repository.

Packaged, validated, and published on a dual Asus GX10 setup. The package's publisher identity is a signing identity, not an authorship claim.
All credits for the model, engine, kernels, and fixes belong to the projects and people listed above.

## License

Weights derive from
[`nvidia/GLM-5.3-Flash-FP4-Local`](https://huggingface.co/nvidia/GLM-5.3-Flash-FP4-Local)
and are subject to the **[NVIDIA Open Model Agreement](https://www.nvidia.com/en-us/agreements/enterprise-software/nvidia-open-model-agreement/)** (license recorded by the
exporter: `nvidia-open-model-agreement`). Review and accept that agreement before use.

Hyperflare is licensed under `Apache-2.0`.