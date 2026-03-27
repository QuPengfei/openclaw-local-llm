
### Introduction

The **OpenClaw local LLM Deployment** is one-click to deploy the openclaw with backend vLLM, llama.cpp

### Build the docker image for OpenClaw+Pinchbench

```
docker build -t pinchbench:v1 -f Dockerfile.1.pinchbench .
```

### Download the models

```
 download the ai models in the folder *models*
```
### Setup

#### 

In the container, node userr with uid=1000
```
sudo chown -R 1000:1000 ./config
sudo chown -R 1000:1000 ./workspace
```

#### Start/stop the openclaw and vLLM

Now it support the following backend and models:

- vLLM on GPU: Qwen3-32B
- llamacpp on CPU: Qwen3.5-35B-A3B-Q8_0.gguf

```
docker compose up -d
docker compose down
```

#### run benchmark with pinchbench

enter into the container
```
docker exec -ti openclaw-pinch bash
```
##### vLLM as backend with Qwen/Qwen3-32B
```
cd /app/skill
./scripts/run.sh --model vllm/Qwen/Qwen3-32B --suite all --no-upload  --judge vllm/Qwen/Qwen3-32B
./scripts/run.sh --model vllm/Qwen/Qwen3-32B --suite automated-only --no-upload
```
##### llama.cpp as backend with 
```
cd /app/skill
./scripts/run.sh --model llamacpp/Qwen3.5-35B-A3B-Q8_0.gguf --suite task_08_memory --no-upload -v
```
#### Open with brower
```
http://ip:18789
```

#### How to verify the openclaw

```
# health check
curl -s http://localhost:18789/healthz
# the expected output {"status":"ok"}

# ready status check
curl -s http://localhost:18789/readyz
# expected output: {"status":"ready"}

# verify with token
curl -s -H "Authorization: Bearer 84a354779b04addc69a7d7aae4dccd4e0faed55742a13e8fc45f67dd23fa2f2d" \
  http://localhost:18789/v1/models
```

### openclaw.json

it is example in the config/openclaw.json

Just remind to replace the llamacpp-service/vllm-service with you LLM server IP

From
"baseUrl": "http://vllm-service:8081/v1"
```
{
  "models": {
    "providers": {
      "vllm": {
        "baseUrl": "http://vllm-service:8081/v1",
        "apiKey": "vllm-local",
        "api": "openai-completions",
        "models": [
          {
            "id": "Qwen/Qwen3-30B-A3B",
            "name": "Qwen/Qwen3-30B-A3B",
            "contextWindow": 32768,
            "maxTokens": 4096
          }
        ]
      }
    }
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "Qwen/Qwen3-30B-A3B"
      },
      "workspace": "/home/node/.openclaw/workspace",
      "compaction": {
        "mode": "safeguard"
      }
    }
  },
  "commands": {
    "native": "auto",
    "nativeSkills": "auto",
    "restart": true,
    "ownerDisplay": "raw"
  },
  "gateway": {
    "port": 18789,
    "mode": "local",
    "bind": "lan",
    "controlUi": {
      "allowedOrigins": [
        "http://localhost:18789",
        "http://127.0.0.1:18789"
      ],
      "allowInsecureAuth": true,
      "dangerouslyDisableDeviceAuth": true
    },
    "auth": {
      "mode": "token",
      "token": "84a354779b04addc69a7d7aae4dccd4e0faed55742a13e8fc45f67dd23fa2f2d"
    }
  }
}
```

### See Also

- [Options]()

