# vLLM + Llama.cpp no Jetson Orin AGX 64GB

Servidores Ollama/lama.cpp com modelos Qwen e Gemma 4 integrados ao OpenCode.

## Modelos Disponíveis

| Modelo | Porta | Container | Tecnologia | Status |
|--------|-------|-----------|------------|--------|
| Qwen3.5-35B-A3B | 8000 | vllm-server | vLLM 0.19.0 | ✅ Ativo |
| Gemma-4-26B Q4 | 8002 | llama-cpp-server | llama.cpp | ✅ Ativo |

## Quick Start

```bash
# Iniciar Qwen (porta 8000)
docker compose -f compose/qwen/docker-compose.yml up -d

# Iniciar Gemma 4 (porta 8002)
docker compose -f compose/llama-cpp/docker-compose.yml up -d

# Verificar containers ativos
docker ps

# Ver logs
docker logs -f vllm-server
docker logs -f llama-cpp-server
```

## Configuração OpenCode

Copiar configuração unificada:
```bash
cp ~/dev/l4t-vllm/opencode.json ~/.config/opencode/opencode.json
```

Dois modelos disponíveis no OpenCode:
- **hoth/Qwen3.5-35B-A3B** - porta 8000 (texto + visão)
- **gemma4/gemma-4-26B-A4B-it-Q4_K_M.gguf** - porta 8002 (texto + visão)

## Especificações

### Qwen3.5-35B-A3B

| Parâmetro | Valor |
|-----------|-------|
| Modelo | `apolo13x/Qwen3.5-35B-A3B-quantized.w4a16` |
| Tipo | Image-Text-to-Text |
| Tamanho | ~20.4 GB (INT4) |
| Context | 131,076 tokens |
| Features | Reasoning, Tool Calling, Visão, Streaming |

### Gemma-4-26B Q4 GGUF

| Parâmetro | Valor |
|-----------|-------|
| Modelo | `ggml-org/gemma-4-26B-A4B-it-GGUF:Q4_K_M` |
| Tipo | Image-Text-to-Text |
| Tamanho | ~16.8 GB (Q4_K_M) |
| Context | 4,096 tokens |
| Tecnologia | llama.cpp (vLLM não suporta GGUF) |
| Features | Reasoning, Tool Calling, Visão, Streaming |

## Decisão Técnica

**Gemma 4 via llama.cpp (não vLLM)**:
- vLLM 0.19.0 não suporta GGUF para Gemma 4 (`Unknown gguf model_type: gemma4`)
- llama.cpp com container `ghcr.io/nvidia-ai-iot/llama_cpp:gemma4-jetson-orin` funciona perfeitamente
- Modelo Q4_K_M (~16.8GB) cabe na memória do Jetson Orin 64GB

## Comandos Úteis

```bash
# Testar Qwen
curl -X POST http://localhost:8000/v1/chat/completions \
  -H "Authorization: Bearer $VLLM_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model": "Qwen3.5-35B-A3B", "messages": [{"role": "user", "content": "Hello"}]}'

# Testar Gemma 4
curl -X POST http://localhost:8002/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model": "gemma-4-26B-A4B-it-Q4_K_M.gguf", "messages": [{"role": "user", "content": "Hello"}]}'

# Listar modelos
curl http://localhost:8000/v1/models
curl http://localhost:8002/v1/models
```

## Referências

- [OpenCode Providers](https://opencode.ai/docs/providers)
- [vLLM Jetson Orin](https://docs.vllm.ai/en/latest/installation/jetson.html)
- [Qwen3.5-35B](https://huggingface.co/apolo13x/Qwen3.5-35B-A3B-quantized.w4a16)
- [Gemma-4-26B GGUF](https://huggingface.co/ggml-org/gemma-4-26B-A4B-it-GGUF)