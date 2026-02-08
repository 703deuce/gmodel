# GLM-OCR RunPod Serverless Worker

Run [zai-org/GLM-OCR](https://huggingface.co/zai-org/GLM-OCR) on RunPod Serverless via vLLM’s OpenAI-compatible API. Only the core files in this repo are committed; vLLM and GLM-OCR are pulled at build/runtime.

## Structure

- **Dockerfile** – GPU image based on `vllm/vllm-openai:nightly` (CUDA + PyTorch included).
- **handler.py** – RunPod worker: starts the vLLM OpenAI server on cold start, then proxies jobs to `/v1/chat/completions`.
- **requirements-runpod.txt** – Python deps (runpod, vLLM, transformers, etc.).
- **README.md** – This file.

## Build

```bash
docker build -t glmocr-runpod .
```

## Run locally (optional)

```bash
docker run --gpus all -p 8080:8080 glmocr-runpod
```

## RunPod job input

Send a job with `input` like:

```json
{
  "input": {
    "prompt": "Text Recognition:",
    "image_url": "/input/image.png"
  }
}
```

- **prompt** – Text prompt (default: `"Text Recognition:"`).
- **image_url** – Path or URL to the image (e.g. RunPod volume path or public URL). Must be reachable from the container.

## Response

```json
{
  "output": "<recognized text from GLM-OCR>"
}
```

## Environment

- **VLLM_PORT** – Port for vLLM (default: `8080`).
- **VLLM_MODEL** – Model name (default: `zai-org/GLM-OCR`).

## Notes

- First request after cold start waits for the vLLM server to load the model; subsequent requests use the same process.
- If the GLM-OCR recipe recommends a specific vLLM version, pin it in `requirements-runpod.txt` (e.g. `vllm==0.6.2`).
