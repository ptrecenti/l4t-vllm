# Agente: Configuração vLLM + OpenCode no Jetson Orin

## Objetivo
Configurar um servidor vLLM via Docker Compose para servir o modelo `apolo13x/Qwen3.5-35B-A3B-quantized.w4a16` em hardware Jetson Orin, integrando com o editor OpenCode.

## Modelo
- **Nome:** `apolo13x/Qwen3.5-35B-A3B-quantized.w4a16`
- **Tipo:** Image-Text-to-Text (suporta texto + imagens como input)
- **Tamanho:** ~20.4 GB (quantizado INT4 via GPTQ)
- **Recuperação de accuracy:** 99.5% em relação ao modelo original
- **Context length:** 131,076 tokens
- **Referência:** [HuggingFace - apolo13x/Qwen3.5-35B-A3B-quantized.w4a16](https://huggingface.co/apolo13x/Qwen3.5-35B-A3B-quantized.w4a16)

## Features Configuradas

### vLLM Server
- [x] Imagem Docker: `ghcr.io/nvidia-ai-iot/vllm:latest-jetson-orin`
- [x] GPU access via NVIDIA Container Toolkit
- [x] Max model len: 131,076 tokens
- [x] GPU memory utilization: 0.70 (70%)
- [x] Reasoning parser: `qwen3`
- [x] Tool calling: `--enable-auto-tool-choice --tool-call-parser qwen3_coder`
- [x] Prefix caching habilitado
- [x] Multimodal encoder TP mode: `data`
- [x] Processor cache: SHM
- [x] HF_TOKEN configurado via ambiente
- [x] **API Key Authentication** - Bearer token protection

### OpenCode Integration
- [x] Provider: `@ai-sdk/openai-compatible`
- [x] Base URL: `http://localhost:8000/v1`
- [x] Modalities configuradas: `["text", "image"]` input, `["text"]` output
- [x] API key via `~/.local/share/opencode/auth.json`
- [x] Model integrado e funcionando

## Testes Realizados

| Feature | Status | Resultado |
|---------|--------|-----------|
| Texto | ✅ | Respostas corretas |
| Visão | ✅ | Descreveu imagens corretamente |
| Reasoning | ✅ | Pensamento passo-a-passo |
| Tool Calling | ✅ | Funções executadas |
| Streaming | ✅ | SSE funcionando |
| OpenCode Integration | ✅ | Modelo listando e respondendo |

## Problemas Resolvidos

### Issue #20802 - Custom OpenAI-compatible providers: image attachments
**Problema:** OpenCode não enviava imagens para providers custom via `@ai-sdk/openai-compatible`.

**Solução:** Adicionar propriedade `modalities` na configuração do modelo:
```json
"modalities": {
  "input": ["text", "image"],
  "output": ["text"]
}
```

**Referência:** [GitHub Issue #20802](https://github.com/anomalyco/opencode/issues/20802)

### HF_HOME Override
**Problema:** Container usava `HF_HOME=/data/models/huggingface` que sobrescrevia o volume mount.

**Solução:** Adicionar variável de ambiente explícita:
```yaml
- HF_HOME=/root/.cache/huggingface
- TRANSFORMERS_CACHE=/root/.cache/huggingface
```

### Nome do Modelo
**Problema:** Modelo original (`Kbenkhaled/...`) não disponível.

**Solução:** Usar substituto funcional `apolo13x/Qwen3.5-35B-A3B-quantized.w4a16`

## Recursos Adicionais

- [Coding Agent with Self-hosted LLM: End-to-End Control with Opencode and vLLM](https://cefboud.com/posts/coding-agent-self-hosted-llm-opencode-vllm/) - Tutorial de Moncef Abboud
- [OpenCode Providers Documentation](https://opencode.ai/docs/providers)
- [vLLM Qwen3.5 Documentation](https://docs.vllm.ai/)

## Comandos Úteis

```bash
# Gerar nova API key
openssl rand -hex 32

# Iniciar servidor
cd ~/dev/l4t-vllm && docker compose up -d

# Ver logs
docker logs -f vllm-server

# Verificar status (com API key)
curl -H "Authorization: Bearer SUA_API_KEY" http://localhost:8000/v1/models

# Testar inferência (com API key)
curl -X POST http://localhost:8000/v1/chat/completions \
  -H "Authorization: Bearer SUA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model": "Qwen3.5-35B-A3B", "messages": [{"role": "user", "content": "Hello"}]}'

# Copiar config OpenCode
cp ~/dev/l4t-vllm/opencode.json ~/.config/opencode/opencode.json

# Configurar API key no OpenCode (Opção 1: via /connect)
opencode
/connect  # Procure por "Other" e digite "vllm"

# Configurar API key no OpenCode (Opção 2: editar auth.json)
cat >> ~/.local/share/opencode/auth.json << 'EOF'
{
  "vllm": {
    "type": "api",
    "key": "SUA_API_KEY_AQUI"
  }
}
EOF
```
