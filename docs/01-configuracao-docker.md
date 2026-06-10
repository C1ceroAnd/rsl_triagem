# 01 — Configuração do n8n com Docker

## Pré-requisitos

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) instalado e em execução
- [Docker Compose](https://docs.docker.com/compose/) disponível (já incluído no Docker Desktop)

Verifique a instalação:

```bash
docker --version
docker compose version
```

---

## Subindo o n8n

Na raiz do repositório, execute:

```bash
docker compose up -d
```

O Docker vai baixar a imagem do n8n na primeira execução (~500 MB) e iniciar o container em segundo plano.

Acesse o n8n em: **http://localhost:5678**

Na primeira abertura, crie uma conta local (e-mail e senha). Essa conta é apenas local — não se conecta a nenhum servidor externo.

---

## Comandos úteis

```bash
# Ver se o container está rodando
docker ps

# Acompanhar os logs em tempo real
docker logs -f n8n_tcc

# Parar o n8n
docker compose down

# Reiniciar o n8n
docker compose restart

# Parar e remover o container (os dados do volume são preservados)
docker compose down
```

---

## Importar o workflow

1. Acesse **http://localhost:5678**
2. No menu lateral, vá em **Workflows**
3. Clique em **Add workflow → Import from file**
4. Selecione o arquivo `workflows/FLUXO_TCC_OLLAMA.json`

O workflow será importado com todos os nós e conexões. As credenciais (Ollama e Google Sheets) precisarão ser configuradas separadamente — veja os documentos `02` e `03`.

---

## Sobre os dados persistidos

O volume `n8n_data` armazena:

- Workflows salvos
- Credenciais configuradas (criptografadas)
- Histórico de execuções

Os dados ficam em um volume gerenciado pelo Docker e **não são apagados** ao reiniciar ou atualizar o container. Para localizar o volume no disco:

```bash
docker volume inspect n8n_tcc_n8n_data
```

> **Importante:** não suba o conteúdo do volume para o repositório. Ele contém chaves de credenciais. O `.gitignore` já exclui os caminhos relevantes.
