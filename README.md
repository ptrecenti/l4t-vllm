# vLLM + OpenCode no Jetson Orin

Servidor vLLM com Qwen3.5-35B-A3B quantizado integrado ao OpenCode para edição de código com IA.

## Quick Start

```bash
# 1. Copiar token HF (se não tiver)
export HF_TOKEN=seu_token_aqui

# 2. Iniciar servidor vLLM
cd ~/dev/l4t-vllm && docker compose up -d

# 3. Aguardar download do modelo (~21GB) e inicialização
docker logs -f vllm-server

# 4. Copiar config OpenCode
mkdir -p ~/.config/opencode
cp ~/dev/l4t-vllm/opencode.json ~/.config/opencode/opencode.json

# 5. Usar no OpenCode
opencode --model vllm/Qwen3.5-35B-A3B
```

## Especificações

| Parâmetro | Valor |
|-----------|-------|
| Modelo | `apolo13x/Qwen3.5-35B-A3B-quantized.w4a16` |
| Tamanho | ~20.4 GB (INT4) |
| Context | 131,076 tokens |
| Input | Texto + Imagens |
| Output | Texto |
| GPU Memory | 70% |

## Features

- ✅ **Reasoning** - Pensamento passo-a-passo (`qwen3` parser)
- ✅ **Tool Calling** - Execução de funções (`qwen3_coder`)
- ✅ **Visão** - Análise de imagens
- ✅ **Prefix Caching** - Cache de prefixos para eficiência
- ✅ **Streaming** - Respostas em tempo real via SSE

## Configurações

### docker-compose.yml

```yaml
services:
  vllm-server:
    image: ghcr.io/nvidia-ai-iot/vllm:latest-jetson-orin
    environment:
      - HF_HOME=/root/.cache/huggingface
      - VLLM_ALLOW_LONG_MAX_MODEL_LEN=1
    command:
      - --model apolo13x/Qwen3.5-35B-A3B-quantized.w4a16
      - --max-model-len 131076
      - --gpu-memory-utilization 0.70
      - --reasoning-parser qwen3
      - --enable-auto-tool-choice
      - --tool-call-parser qwen3_coder
      - --enable-prefix-caching
```

### opencode.json

```json
{
  "provider": {
    "vllm": {
      "npm": "@ai-sdk/openai-compatible",
      "options": { "baseURL": "http://localhost:8000/v1" },
      "models": {
        "Qwen3.5-35B-A3B": {
          "modalities": { "input": ["text", "image"], "output": ["text"] }
        }
      }
    }
  }
}
```

## Testes

```bash
# Teste de texto
curl -X POST http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model": "Qwen3.5-35B-A3B", "messages": [{"role": "user", "content": "Hello"}]}'

# Teste via OpenCode
echo 'Say "Hello from vLLM"' | opencode run --model vllm/Qwen3.5-35B-A3B
```

## Notas

- Modelo requer ~21GB de download na primeira execução
- Monta cache HuggingFace do host para persistência
- Para áudio/vídeo, seria necessário modelo VL completo (Qwen3.5-VL)

## Referências

- [Modelo no HuggingFace](https://huggingface.co/apolo13x/Qwen3.5-35B-A3B-quantized.w4a16)
- [vLLM Jetson Orin Image](https://ghcr.io/nvidia-ai-iot/vllm:latest-jetson-orin)
- [OpenCode Providers](https://opencode.ai/docs/providers)
