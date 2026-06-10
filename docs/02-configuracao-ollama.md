# 02 — Configuração do Ollama

O Ollama roda diretamente no macOS (fora do Docker) e serve o modelo `llama3.2:3b` para o n8n via HTTP.

---

## Instalação

Baixe e instale o Ollama em: **https://ollama.com/download**

Ou via Homebrew:

```bash
brew install ollama
```

---

## Baixar o modelo

```bash
ollama pull llama3.2:3b
```

O download é de aproximadamente 2 GB. Execute uma vez — o modelo fica armazenado localmente em `~/.ollama/models`.

---

## Iniciar o servidor

```bash
ollama serve
```

O Ollama ficará disponível em: **http://localhost:11434**

Para verificar se está rodando:

```bash
curl http://localhost:11434/api/tags
```

A resposta deve listar `llama3.2:3b` entre os modelos disponíveis.

> O Ollama precisa estar rodando **antes** de executar qualquer triagem no n8n. Se o Mac for reiniciado, execute `ollama serve` novamente.

---

## Configurar a credencial no n8n

O n8n roda dentro do Docker e precisa alcançar o Ollama no host do Mac. O endereço correto para isso é `host.docker.internal` — um DNS especial que o Docker resolve automaticamente para o IP da máquina host.

**Passo a passo:**

1. Acesse **http://localhost:5678**
2. No menu lateral, vá em **Credentials**
3. Clique em **Add credential**
4. Busque por **Ollama** e selecione
5. Preencha o campo **Base URL** com:

```
http://host.docker.internal:11434
```

6. Clique em **Save**

---

## Vincular a credencial ao workflow

1. Abra o workflow importado
2. Clique no nó **Ollama Chat Model**
3. No campo **Credential**, selecione a credencial Ollama recém-criada
4. Confirme que o modelo está definido como `llama3.2:3b`
5. Confirme que **Temperature** está em `0`

---

## Comandos úteis do Ollama

```bash
# Listar modelos baixados
ollama list

# Testar o modelo diretamente no terminal
ollama run llama3.2:3b

# Ver logs do servidor
ollama serve 2>&1 | head -50
```
