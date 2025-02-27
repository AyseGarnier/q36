<p align="center">
  <img src="logo.svg" alt="QuarkStar logo" width="220">
</p>

**QuarkStar** is a small native inference engine for **Qwen3.6-35B-A3B**,
**Qwen3.8-27B**.
It is self-contained and deliberately narrow, not a general GGUF runner. 
The main paths are `qwen35moe`-specific Vulkan and Metal graph executors with
Q36-specific loading, prompt rendering, tool calls, KV state, HTTP server,
and coding agent. The repository also includes tools and data for GGUF,
imatrix, quality, and speed.

This project would not exist without **DwarfStar**, **llama.cpp and GGML**, make
sure to read the acknowledgements section, a big thank you to Salvatore Sanfilippo 
aka Antirez and Georgi Gerganov and all the other contributors.

Model support is intentionally opportunistic. The project follows the best open
weights for useful local machine sizes, especially 16 GB machines and 24/32 GB
workstations. A model may be removed when a better replacement arrives.

Supported runtimes:

* **Metal** on Apple Silicon M1 or newer. A 16 GB Mac can run the default
  release model resident; 8 GB Macs are intended to use SSD streaming.
* **Vulkan** on Linux. The backend is generic; its current fast path was
  developed and tuned on the AMD BC-250 with RADV.
* **CPU** as a portable reference and debugging runtime, not the normal
  production path.

# So, what can I do with this software?

* You can run a quite capable model on your very cheap hardware, a $150 machine, 
the BC-250. Even if you don't have enough RAM, with SSD streaming, you can still 
run it at a decent speed.
* You can run the same CLI, agent, evaluation harness, benchmark, and server on
  Apple Silicon through the native Metal graph runtime.

## Requirements

QuarkStar has two native GPU backends:

* **Linux with Vulkan 1.1 or newer** and support for shader `float64` and
  `int64`. Subgroup arithmetic, 16-bit storage, and native `float16` are
  detected at runtime; optimized kernels fall back when an optional feature
  is unavailable.
* **AMD BC-250** (Cyan Skillfish, 24 CUs, 16 GB unified GDDR6) is the primary
  tested Vulkan device. Its vendor/device ID selects the tuned fast path
  automatically. Follow [BC250.md](BC250.md) for the RADV, UMA, kernel-memory,
  governor, build, and optional 40-CU setup.
* **Apple Silicon M1 or newer** with macOS 11 or newer and Xcode or the Command
  Line Tools providing the macOS SDK and Metal framework. Build with
  `make metal` and run with `./q36 --metal`. Shader sources are compiled through
  Metal at runtime, so the build does not require a separate offline
  `metal` invocation. Metal builds use an Apple Silicon/macOS 11 deployment
  target; newer residency APIs are selected only at runtime.

Windows is not currently a build target. The Metal backend uses unified-memory,
mmap-backed model buffers and is independent of the Vulkan build.

## Motivations

* Small open-weight models are already good and fit on normal personal machines, 
and they'll keep getting better.
* AI providers' flat plans keep raising prices, and not everyone can afford $3k–$5k machines.
* BC-250 is the perfect machine for that: $150, a powerful GPU, fast memory, 
decent SSD speed support, and unified memory.
* Qwen3.6-35B-A3B tolerate aggressive routed-expert quantization (recipe by Antirez).
* Qwen3.6-35B-A3B is fast as hell and can potentially run even on a toaster, Vulkan 
is compatible on paper with a huge range of devices, and with DZN and Lavapipe it 
opens up some very interesting possibilities in the future.
* Compressed KV caches and fast local SSDs make long contexts practical.
* The idea of an inference system specialized for a few models.

# AI full disclosure

* This software is developed with **strong assistance from GPT 5.5, 5.6, Claude Fable**
 and with humans leading the ideas, testing, and debugging. We say this openly because 
 it shaped how the project was built. If you are not happy with AI-developed code, this 
 software is not for you. The acknowledgement below is equally important: this would not 
 exist without `llama.cpp` and GGML, largely written by hand.

## Acknowledgements

### To antirez and ds4

QuarkStar is essentially a port of [DwarfStar](https://github.com/antirez/ds4),
redesigned for the Vulkan runtime and retargeted at Qwen3.6-35B-A3B. The Vulkan
engine adapts DwarfStar's ideas to a different model and device. The Apple
runtime also retains and specializes DwarfStar's mature Metal kernel library
for Qwen3.6, including the routed IQ2_XXS/Q2_K expert path.

**Special thanks to Salvatore**, he is a continuous source of inspiration for
me, and his content on YouTube has greatly improved me as a software engineer
and as a person.

This project was born with the intent of improving my skills in LLMs. It's
useful for me for inference and for learning, and I hope it will be useful for
you too.

### To llama.cpp and GGML

`q36.c` does not link against GGML, but it **exists thanks to the path opened by the
llama.cpp project and the kernels, quantization formats, GGUF ecosystem, and hard-won
engineering knowledge developed there**.
We are thankful and indebted to [`llama.cpp`](https://github.com/ggml-org/llama.cpp)
and its contributors. Their implementation, kernels, tests, and design choices were
an essential reference while building this Qwen3.6-35B-A3B specific inference path.
Some source-level pieces are retained or adapted here under the MIT license: GGUF
quant layouts and tables, CPU quant/dot logic, and certain kernels. For this
reason, and because we are genuinely grateful, we keep the GGML authors copyright
notice in our `LICENSE` file.

## Status

The software is currently very fast changing. Consider it alpha quality.
Before each release, a big QA run is executed, however instabilities
are definitely possible.

## More Documentation

The focused documents below cover development, release checks, model details,
and offline tooling. For normal usage, keep reading the next sections.

- [CONTRIBUTING.md](CONTRIBUTING.md): correctness and speed regression testing
  guide for contributors. **Read this before sending a pull request**.
- [BC250.md](BC250.md): Linux, RADV, unified-memory, governor, and build setup
  for the primary Vulkan device.
- [QA_BEFORE_RELEASES.md](QA_BEFORE_RELEASES.md): the complete release test
  matrix.
- [MODEL_CARD.md](MODEL_CARD.md): the fixed Qwen3.6 architecture, tokenizer,
  quantization, and sampling assumptions used by Q36.
- [gguf-tools/README.md](gguf-tools/README.md): offline GGUF generation,
  imatrix collection, quantization tooling, and quality checks.
- [gguf-tools/imatrix/dataset/README.md](gguf-tools/imatrix/dataset/README.md):
  how the calibration prompt corpus is generated.
- [gguf-tools/quality-testing/README.md](gguf-tools/quality-testing/README.md):
  how local GGUFs are scored against OpenRouter continuations.
- [dir-steering/README.md](dir-steering/README.md): the preserved DS4 vector
  generation experiment that informed Q36's directional-steering runtime.
- [tests/test-vectors/README.md](tests/test-vectors/README.md): tracked
  llama.cpp continuation vectors used for regression checks.

## Model Weights

This implementation works with Qwen3.6-35B-A3B and dense Qwen3.8-27B GGUFs.
It is not a general GGUF loader, and arbitrary GGUF files will not have
the tensor layout, quantization mix, metadata, or optional MTP state expected by
the engine. The 2 bit quantizations provided here are verified to be actually
high quality: they behave well, work under coding agents, call tools in a reliable way.

Start dense Qwen3.8-27B explicitly:

```sh
./q36-server \
  -m gguf/Qwen3.8-27B-UD-IQ3_S.gguf \
  --vulkan
```

### Vision

Q36 accepts the matching Qwen3-VL projector as a disk-backed sidecar. It maps
the file without copying the model and streams one weight tensor at a time
through a reusable Vulkan buffer. The full 0.9 GiB projector is never retained
in RAM or VRAM, and streamed pages are released after each dispatch.

```sh
./q36 --vision gguf/Qwen3.6-35B-A3B-mmproj-F16.gguf
./q36 -m gguf/Qwen3.8-27B-UD-IQ3_S.gguf \
  --vision gguf/Qwen3.8-27B-mmproj-F16.gguf
```

At the interactive prompt, `/read photo.jpg` and `/read image.png` submit an
image turn. An optional prompt can be passed along with the image (e.g.
`/read photo.jpg Describe this image` or `/read "path/with spaces/img.png" What is here?`).
JPEG and PNG decoding is built in. The sidecar output dimension is
validated against the selected standard or dense language model before use.
Vision prompt execution currently requires Vulkan. `q36-agent --vision FILE`
exposes `view_image` for local files. `q36-server --vision FILE` accepts OpenAI
image URLs containing PNG/JPEG data URIs, Responses image blocks, and Anthropic
base64 images, including images returned by tools. Remote image URLs are rejected.
The server caches at most 32 MiB of decoded image embeddings and encoded keys;
the projector weights remain streamed from disk. Prefix reuse checks the image
fingerprints and geometry, so replacing pixels with the same number of image
pads rebuilds the context. Vision sessions currently cannot be saved as agent
sessions or disk KV checkpoints.

The 2 bit quants use a very asymmetrical quantization: only the routed MoE
experts are quantized, up/gate at `IQ2_XXS`, down at `Q2_K`. They are the
majority of all the model space: the other components (shared experts,
projections, routing) are left untouched to guarantee quality. The resulting
weight footprint is roughly 10-11 GB, which fits the BC-250 with room left
for KV cache and OS.

The higher-quality mixed model keeps that layout for layers 0..33 and uses
`Q4_K` routed gate, up, and down tensors in layers 34..39. Its roughly 13 GB
weight file is intended for SSD streaming on 16 GB machines.

Download one main model.

```sh
./download_model.sh q2-imatrix   # 16 GB unified memory machines
./download_model.sh q2-q4-imatrix # higher quality; stream on 16 GB machines
./download_model.sh 27b           # Dense Qwen3.8-27B IQ3_S
```

Choose one. The first target matches the compiled default path. Select the
second explicitly:

```sh
./q36 -m gguf/Qwen3.6-35B-A3B-Layers34-39Q4KExperts-OtherExpertLayersIQ2XXSGateUp-Q2KDown-Q8Rest-imatrix.gguf \
  --ssd-streaming -p "Hello"
```

The script downloads Qwen3.6 models from
`https://huggingface.co/Ninnix96/Qwen3.6-35B-A3B-gguf` and dense Qwen3.8-27B
from `https://huggingface.co/unsloth/Qwen3.8-27B-GGUF`. It stores files under
`./gguf/`, resumes partial downloads with `curl -C -`, and updates
`./q36moe.gguf` to point at the selected model for older scripts.

Then build for the target platform:

```sh
make                  # Linux / generic Vulkan, automatic BC-250 fast path
make vulkan-generic   # explicit generic Vulkan build
make vulkan-bc250     # Vulkan build that requires a BC-250 at runtime
make metal            # macOS / Apple Silicon
```

The normal and CPU-only builds are self-contained. They do not require a
llama.cpp checkout or GGML libraries; those are used only by explicit optional
reference targets under `make test-llama` and `make test-vectors-local`.

The vendored JPEG/PNG decoder in `third_party/iris` is Copyright (c) 2026
Salvatore Sanfilippo and distributed under its included MIT license.

`gguf/Qwen3.6-35B-A3B-AntirezExperts-IQ2XXS-gateup-Q2K-down-Q8rest.gguf` is
the default model path used by all runtime binaries. Pass `-m` to select another
supported GGUF from `./gguf/`. Run `./q36 --help` and
`./q36-server --help` for the full flag list.

If you want to regenerate GGUF files, quantize community fine-tunes / abliterated models
(via `--strip-nextn`), or collect a new imatrix, see
[gguf-tools/README.md](gguf-tools/README.md). Those tools are meant for offline
Qwen3.6 model-building work. The native quantizer accepts Q8, F16, or BF16
inputs; imatrix collection still uses the optional llama.cpp tooling.

`./download_model.sh mtp` fetches the optional speculative decoding support
GGUF for Qwen 3.6 MoE. It can be used with the `q2-imatrix` and
`q2-q4-imatrix` main models, but must be enabled explicitly with `--mtp`. The
MTP block can also be embedded in a 41-block MoE GGUF. Pass the same path to
`-m` and `--mtp` to use it. With `--mtp` omitted, q36 runs the 40-block trunk
and excludes block 40 from the resident GPU weight cache. The embedded form
keeps the shared embedding and output tensors in one copy.

The current MTP/speculative decoding path is still experimental: it is
correctness-gated and currently provides at most a slight speedup, not a
meaningful generation-speed win.

## Speed

Benchmarks on a **BC-250** (40 CUs unlocked, Cyan-Skillfish Governor performance
mode at 500-2000 MHz, 85 °C thermal ceiling — a hot summer in Italy). Runs
use greedy decoding, `--gen-tokens 128`, the extended long-context story
prompt, and Q8_0 K / Q4_0 V cache. The q2 model is fully resident. The mixed
q2-q4 model uses SSD streaming with the automatic 5724-expert (4.72 GiB)
cache. Each row was measured in a separate process, with only one benchmark
process running at a time.

| Machine | Quant | Mode | Prompt | Prefill | Generation |
| --- | ---: | --- | ---: | ---: | ---: |
| BC-250 (40 CU) | q2 | Resident | 2048 ctx | 639.85 t/s | 81.85 t/s |
| BC-250 (40 CU) | q2 | Resident | 4096 ctx | 597.10 t/s | 79.74 t/s |
| BC-250 (40 CU) | q2 | Resident | 8192 ctx | 501.50 t/s | 74.72 t/s |
| BC-250 (40 CU) | q2 | Resident | 16384 ctx | 373.32 t/s | 65.04 t/s |
| BC-250 (40 CU) | q2 | Resident | 24576 ctx | 287.24 t/s | 56.17 t/s |
| BC-250 (40 CU) | q2 | Resident | 32768 ctx | 244.06 t/s | 51.26 t/s |
| BC-250 (40 CU) | q2-q4 | SSD streaming | 4096 ctx | 45.43 t/s | 14.52 t/s |
| BC-250 (40 CU) | q2-q4 | SSD streaming | 8192 ctx | 48.78 t/s | 13.98 t/s |

![BC-250 Q2 t/s](speed-bench/bc250_ts.svg)

### Metal measurements

The following single-process measurements used the default q2 GGUF fully
resident and the mixed q2-q4 GGUF with SSD streaming on a 16 GB M2 Pro with
macOS 26.5. The SSD run used the automatic 4240-expert (6.99 GiB) cache. Both
runs used `tests/long_context_story_prompt.txt`, incremental prefill at doubling
context frontiers, greedy decoding with 128 generated tokens per frontier, and
Q8_0 K / Q4_0 V cache:

| Runtime | Quant | Expert residency | Prompt | Prefill | Generation |
| --- | ---: | --- | ---: | ---: | ---: |
| Metal | q2 | Full model resident | 2048 ctx | 448.75 t/s | 37.78 t/s |
| Metal | q2 | Full model resident | 4096 ctx | 366.19 t/s | 34.93 t/s |
| Metal | q2 | Full model resident | 8192 ctx | 270.02 t/s | 31.08 t/s |
| Metal | q2 | Full model resident | 16384 ctx | 177.21 t/s | 25.64 t/s |
| Metal SSD | q2-q4 | 4240 expert slots / 6.99 GiB | 4096 ctx | 58.74 t/s | 8.97 t/s |
| Metal SSD | q2-q4 | 4240 expert slots / 6.99 GiB | 8192 ctx | 55.32 t/s | 9.20 t/s |

![M2 Pro Metal Q2 t/s](speed-bench/metal_q2_ts.svg)

These numbers show the expected capacity tradeoff, not a cross-machine
performance promise. Metal generation, prefill, and SSD behavior should be
remeasured on the oldest and smallest supported Mac before a release.

Use `q36-bench` for reproducible prefill and decode measurements. Release
builds also have a conservative BC-250 performance gate under
`make benchmark-gate`; record results on the same board and power state when
comparing changes.

## Running Models Larger Than Available Memory

Metal and Vulkan SSD streaming keep non-routed weights resident and load
selected routed experts into a bounded backend cache. The planner reserves memory
for the configured context, prefill scratch, planned server sessions, and a staging
margin before assigning the remaining model budget to static weights and experts.
Explicit cache sizes are targets capped by the same budget. Creating a larger or
additional session reduces the expert cache when necessary; it fails cleanly if
static weights and the minimum working cache leave insufficient room. Startup
logs show the resolved context and model budgets.

```sh
./q36 --metal --ssd-streaming -p "Explain radix trees."
./q36 --vulkan --ssd-streaming --ssd-streaming-cache-experts 256 -p "Hello"
./q36 --metal --ssd-streaming --ssd-streaming-cache-experts 1GB -p "Hello"
```

`NGB` is a routed-expert byte budget. Q36 converts it to the number of expert
slots that fit the release GGUF. A plain integer requests an expert-slot count;
both forms are capped to leave room for the configured contexts.
Non-routed weights, KV cache, activations, and graph scratch need additional
memory. Startup prints the resolved slot count and actual cache allocation.

A built-in or `Q36_VK_STREAMING_EXPERT_HOTLIST` profile biases eviction
without reading expert weights at startup. It does not fill the cache with
arbitrary experts when no profile exists. Use `--ssd-streaming-cold` for an
empty cache, or `--ssd-streaming-preload-experts N` to request an explicit
weight preload. Metal stores mixed expert sizes in component-wise padded cache
slots, so the IQ2/Q2 and Q4 routed layers share one bounded cache and one
hotlist policy. Vulkan retains its original single-size-class behavior. The
`Q36_VK_STREAMING_EXPERT_HOTLIST` and
`Q36_VK_DISABLE_STREAMING_EXPERT_HOTLIST` names are retained for compatibility
and apply to both graph runtimes.

As in DS4, an explicit routed prefix can stay fully resident:

```sh
./q36 --ssd-streaming --ssd-streaming-full-layers 4 -p "Hello"
```

Full layers are charged at their actual byte size and the remaining budget
must still hold one layer of dynamic expert slots for prefill. The default is
zero because the dynamic-only cache is faster on the BC-250. Pass
`--ssd-streaming-full-layers 0` to disable an explicit setting.

### Metal SSD streaming

Metal follows DS4's bounded expert-cache design. Routed expert bytes are read
from the GGUF into size-limited shared Metal buffers, selected IDs are remapped
to cache slots, and cache misses use a biased least-recently-used eviction
policy. `--ssd-streaming-cache-experts`, preload, cold-cache, and
full-resident-layer controls therefore allocate and constrain real Metal
buffers; they are not hints to macOS VM paging.

For the mixed 13 GB model, each dynamic slot is sized for the largest routed
tensor layout. Smaller IQ2/Q2 experts and the six Q4_K expert layers can remain
cached together; changing precision at layer 34 does not reset the cache or
fall back to the CPU.

On a 16 GB Mac, start with the resident q2 model:

```sh
./q36 --metal -p "Hello"
```

For the larger release GGUF, or on an 8 GB Mac, start conservatively:

```sh
./q36 --metal --ssd-streaming --ctx 4096 -p "Hello"
./q36 --metal --ssd-streaming \
  --ssd-streaming-cache-experts 512 --ctx 4096 -p "Hello"
```

The automatic form uses `recommendedMaxWorkingSetSize`. The explicit example
allocates about 0.42 GiB for the q2 model or 0.85 GiB for the mixed model,
whose slots are padded for Q4_K. It leaves more headroom for the OS and KV
state. Increase it only while memory pressure, swap, and responsiveness remain
acceptable. A physical 8 GB Apple Silicon run is part of release QA;
`--simulate-used-memory 8GB` on a larger Mac is diagnostic evidence, not a
replacement.

`--ssd-streaming-preload-experts N` performs an actual startup read. The normal
automatic hotlist only biases eviction and does not read expert weights.
`--ssd-streaming-full-layers N` loads the first `N` routed layers into separate
full-resident Metal buffers and deducts their exact bytes from the dynamic
budget. `q36-bench` prints cache slots, cache/full-layer GiB, hits, misses,
loads, evictions, and GGUF bytes read in its final memory report.

## Native Agent

Q36 includes a native coding agent. Inference is controlled inside the agent
itself, without a socket or API boundary, so the transcript and live KV state
are one session. The tools and system prompt use Qwen3.6's native tagged tool
format directly. This provides a few advantages:

- Low latency for generated text, tool calls, and new sessions.
- Live progre