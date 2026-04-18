# infer-job-config

Configuration templates for running inference jobs with popular open-source
language models.  Each template provides ready-to-use settings for
[vLLM](https://docs.vllm.ai), [Hugging Face TGI](https://huggingface.co/docs/text-generation-inference),
and Kubernetes / container deployments.

---

## Directory structure

```
configs/
├── base-template.yaml          # Annotated template – start here
└── models/
    ├── llama/
    │   ├── llama2-7b.yaml
    │   ├── llama2-13b.yaml
    │   ├── llama2-70b.yaml
    │   ├── llama3-8b.yaml
    │   └── llama3-70b.yaml
    ├── mistral/
    │   ├── mistral-7b-instruct.yaml
    │   ├── mixtral-8x7b-instruct.yaml
    │   └── mixtral-8x22b-instruct.yaml
    ├── falcon/
    │   ├── falcon-7b-instruct.yaml
    │   ├── falcon-40b-instruct.yaml
    │   └── falcon-180b.yaml
    ├── phi/
    │   ├── phi-3-mini-4k-instruct.yaml
    │   ├── phi-3-mini-128k-instruct.yaml
    │   └── phi-3-medium-128k-instruct.yaml
    ├── gemma/
    │   ├── gemma-2b-it.yaml
    │   ├── gemma-7b-it.yaml
    │   ├── gemma-2-9b-it.yaml
    │   └── gemma-2-27b-it.yaml
    ├── qwen/
    │   ├── qwen2-7b-instruct.yaml
    │   └── qwen2-72b-instruct.yaml
    └── deepseek/
        ├── deepseek-r1-distill-qwen-7b.yaml
        └── deepseek-r1-distill-llama-70b.yaml
```

---

## Quick start

1. Pick the config that matches your model and copy it:

   ```bash
   cp configs/models/llama/llama3-8b.yaml my-job.yaml
   ```

2. Edit `my-job.yaml` to set your Hugging Face token, desired `max_model_len`,
   resource limits, etc.

3. Launch with **vLLM**:

   ```bash
   python -m vllm.entrypoints.openai.api_server \
     --model  "$(yq '.model.hf_model_id' my-job.yaml)" \
     --dtype  "$(yq '.inference.dtype' my-job.yaml)" \
     --max-model-len "$(yq '.inference.max_model_len' my-job.yaml)" \
     --tensor-parallel-size "$(yq '.inference.tensor_parallel_size' my-job.yaml)" \
     --gpu-memory-utilization "$(yq '.vllm.gpu_memory_utilization' my-job.yaml)" \
     --port   "$(yq '.serving.port' my-job.yaml)"
   ```

4. Launch with **Hugging Face TGI**:

   ```bash
   docker run --gpus all \
     -e MODEL_ID="$(yq '.model.hf_model_id' my-job.yaml)" \
     -e NUM_SHARD="$(yq '.tgi.num_shard' my-job.yaml)" \
     -e MAX_NEW_TOKENS="$(yq '.tgi.max_new_tokens' my-job.yaml)" \
     -e MAX_TOTAL_TOKENS="$(yq '.tgi.max_total_tokens' my-job.yaml)" \
     -p 8080:80 \
     ghcr.io/huggingface/text-generation-inference:latest
   ```

---

## Configuration fields

| Section | Key | Description |
|---------|-----|-------------|
| `model` | `hf_model_id` | Hugging Face Hub identifier (`org/repo`) |
| `model` | `hf_revision` | Branch, tag, or commit SHA |
| `hardware` | `min_gpus` | Minimum GPUs required |
| `hardware` | `gpu_memory_gb` | VRAM required per GPU (GiB) |
| `inference` | `dtype` | Compute dtype (`float16`, `bfloat16`, `float32`, `auto`) |
| `inference` | `max_model_len` | Maximum sequence length (prompt + completion) |
| `inference` | `max_num_seqs` | Maximum concurrent sequences |
| `inference` | `tensor_parallel_size` | Number of GPUs for tensor parallelism |
| `inference` | `quantization` | PTQ method (`null`, `awq`, `gptq`, `bitsandbytes`, `fp8`) |
| `vllm` | `gpu_memory_utilization` | Fraction of GPU memory for KV cache (0–1) |
| `tgi` | `max_batch_total_tokens` | Maximum tokens across all sequences in a batch |
| `resources` | `requests` / `limits` | Kubernetes resource requests and limits |
| `serving` | `port` | Port for the OpenAI-compatible API |

See [`configs/base-template.yaml`](configs/base-template.yaml) for the full
list of fields with inline documentation.

---

## License

MIT – see [LICENSE](LICENSE).
