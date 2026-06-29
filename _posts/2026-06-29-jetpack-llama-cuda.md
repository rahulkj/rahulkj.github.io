---
title: "Running llama.cpp GPU server on Jetson (Orin Nano)"
date: 2026-06-29T11:33:32-05:00
description: "Leverage GPU's from the NVIDIA ORIN NANO SUPER"
featured: true
draft: false
toc: true
categories:
  - Technology
tags:
  - llama-server
  - Cuda
  - AI
  - Jetpack
---

# 0. SYSTEM PREREQS

```bash
sudo apt update
sudo apt install -y git cmake build-essential python3 python3-venv python3-pip curl
```

---

# 1. Build llama.cpp (GPU ENABLED) from source

```bash
cd ~
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp
```

## Build with CUDA (Jetson / Orin Super Nano)

```bash
cmake -B build \
  -DGGML_CUDA=ON \
  -DCMAKE_BUILD_TYPE=Release

cmake --build build -j$(nproc)
```

### Verify GPU is visible

```bash
./build/bin/llama-cli --list-devices
```

Expected:

```
CUDA0: Orin ...
```

---

# 2. CREATE MODEL DIRECTORY

```bash
mkdir -p ~/models
```

---

# 3. Installing HUGGINGFACE CLI (IMPORTANT)

```bash
python3 -m venv ~/.venv-hf
source ~/.venv-hf/bin/activate

pip install -U huggingface_hub
```

Login:

```bash
hf auth login
```

If permission error:

```bash
mkdir -p ~/.cache/huggingface
chmod -R 700 ~/.cache/huggingface
```

---

# 4. DOWNLOAD WORKING MODELS (GUARANTEED REPOS)

## ✅ BEST SMALL CODING MODEL (RECOMMENDED FOR ORIN)

### Qwen Coder 3B (BEST STABILITY)

```bash
hf download Qwen/Qwen2.5-Coder-3B-Instruct-GGUF \
  qwen2.5-coder-3b-instruct-q4_k_m.gguf \
  --local-dir ~/models
```

---

## Optional stronger model (if RAM allows)

```bash
hf download bartowski/deepseek-coder-6.7B-instruct-GGUF \
  DeepSeek-Coder-6.7B-Instruct-Q4_K_M.gguf \
  --local-dir ~/models
```

---

# 5. VERIFY MODELS EXIST

```bash
ls -lh ~/models
```

You should see:

```
qwen2.5-coder-3b-instruct-q4_k_m.gguf
```

---

# 6. RUN llama-server MANUALLY (TEST)

## IMPORTANT FIXES FOR YOUR ERRORS:

* set model explicitly
* reduce GPU layers
* increase context for OpenCode

```bash
./build/bin/llama-server \
  --host 0.0.0.0 \
  --port 8080 \
  -m ~/models/qwen2.5-coder-3b-instruct-q4_k_m.gguf \
  -c 16384 \
  -ngl 20
```

### If DeepSeek (6.7B):

```bash
-ngl 10
```

---

# 7. VALIDATE API

```bash
curl http://localhost:8080/v1/models
```

---

### Chat test:

```bash
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [
      {"role":"user","content":"Explain Kubernetes in simple terms"}
    ],
    "temperature": 0.7
  }'
```

⚠️ No API key needed — if you saw:

> Invalid API Key

you were hitting OpenAI proxy mode or wrong endpoint config in OpenCode.

---

# 8. CREATE SYSTEMD SERVICE (PROPER FIX)

## Create service:

```bash
sudo nano /etc/systemd/system/llama-server.service
```

Paste this (NO blanks, fully working):

```ini
[Unit]
Description=llama.cpp GPU Server
After=network.target

[Service]
Type=simple
User=nvidia
WorkingDirectory=/home/nvidia/llama.cpp

ExecStart=/home/nvidia/llama.cpp/build/bin/llama-server \
    -m /home/nvidia/models/qwen2.5-coder-3b-instruct-q4_k_m.gguf \
    --host 0.0.0.0 \
    --port 8080 \
    --ctx-size 16384 \
    --batch-size 128 \
    --ubatch-size 128 \
    --n-gpu-layers 99 \
    --flash-attn auto

Restart=always
RestartSec=5
Environment=CUDA_VISIBLE_DEVICES=0

[Install]
WantedBy=multi-user.target
```

---

# 9. ENABLE SERVICE

```bash
sudo systemctl daemon-reload
sudo systemctl enable llama-server
sudo systemctl start llama-server
```

---

## CHECK LOGS

```bash
journalctl -u llama-server -f
```

---

# 10. FIX YOUR OPENCODE ISSUE (CRITICAL)

Your error:

> context size 2048 too small

Fix:

### You MUST run server with:

```bash
-c 4096
```

or:

```bash
-c 8192
```

---

# 11. OPENAI COMPAT MODE FOR OPCODE

Use this base URL:

```bash
http://localhost:8080/v1
```

NO API KEY REQUIRED.

---

## Example OpenCode config:

```json
{
  "provider": "openai",
  "base_url": "http://localhost:8080/v1",
  "api_key": "dummy",
  "model": "/home/nvidia/models/qwen2.5-coder-3b-instruct-q4_k_m.gguf"
}
```

---

# 12. TROUBLESHOOTING YOU HIT (EXPLAINED)

## ❌ GPU OOM (DeepSeek 6.7B)

Fix:

```bash
-ngl 10
-c 4096
```

Or switch to:

* Qwen 3B (recommended)

---

## ❌ systemd crash loop

Cause:

* bad CLI flag (`--flash-attn` missing value)

Fix:

```bash
--flash-attn auto
```

---

## ❌ hf repo not found

Cause:

* wrong repo name (many “TheBloke” repos renamed/deprecated)

Fix:
Use:

* `Qwen/Qwen2.5-Coder-3B-Instruct-GGUF`
* `bartowski/*`

---

## ❌ hf CLI not found

Fix:

```bash
pip install huggingface_hub
hf auth login
```

---

# 13. BEST STACK that runs well on this small formfactor of NVIDIA

| Component  | Choice                    |
| ---------- | ------------------------- |
| Model      | Qwen2.5-Coder-3B-Instruct |
| Quant      | Q4_K_M                    |
| Context    | 4096                      |
| GPU layers | 20                        |
| Server     | llama-server              |

---