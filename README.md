# RSL Triagem

Sistema de triagem automatizada para Revisões Sistemáticas da Literatura (RSL), desenvolvido como TCC no IFPI — Campus Piripiri (2026).

Recebe arquivos bibliográficos exportados de bases como IEEE Xplore, ACM Digital Library e Scopus, processa cada registro com um agente de IA local e grava os resultados em uma planilha Google Sheets. Tudo roda localmente — n8n via Docker, modelo de linguagem via Ollama, painel web aberto direto no navegador.

> Validação realizada sobre a RSL de referência: Rajapakse et al. (2021), *"Challenges and solutions when adopting DevSecOps"*, Information and Software Technology. 935 registros processados, 54 estudos relevantes conhecidos — **Recall de 92,6% (50/54)**.

---

## Arquitetura

```
painel/index.html  →  n8n (Docker :5678)  →  Ollama (Mac :11434)
                            │
                            ▼
                     Google Sheets
```

Pipeline interno do n8n:

```
Webhook (POST /Triagem_RSL)
    │
    ▼
[Listar Arquivos]      extrai conteúdo binário de cada arquivo recebido
    │
    ▼
[Parser BibTeX/RIS]    detecta base, divide entradas, converte para JSON
    │                  IEEE: }@  |  Scopus/ACM: \n\n@  |  RIS: ER -
    ▼
[Limpeza]              normaliza campos: titulo, autores, ano, doi, resumo, fonte
    │
    ▼
[AI Agent]             llama3.2:3b via Ollama, temperature=0
    │                  retorna: decisao / confianca / justificativa
    ▼
[Organizador]          exclusão determinística por data (JS, não LLM)
    │                  parseia resposta JSON do agente
    ▼
[Google Sheets]        append: titulo, autores, ano, doi, fonte,
                       decisao, confianca, justificativa, validacao_humana
```

---

## Stack

| Componente | Tecnologia |
|---|---|
| Orquestração | n8n (Docker local, porta 5678) |
| Modelo de linguagem | llama3.2:3b via Ollama (Mac local, porta 11434) |
| Interface web | HTML + CSS + JS (arquivo local, sem servidor) |
| Armazenamento | Google Sheets (OAuth2) |
| Infraestrutura | Docker / docker-compose, macOS |

---

## Estrutura do repositório

```
rsl_triagem/
├── README.md
├── docker-compose.yml          # n8n local (comentado)
├── .gitignore
│
├── painel/
│   └── index.html              # Abrir direto no navegador
│
├── workflows/
│   └── FLUXO_TCC_OLLAMA.json   # Importar no n8n
│
└── docs/
    ├── 01-configuracao-docker.md
    ├── 02-configuracao-ollama.md
    ├── 03-configuracao-google-sheets.md
    └── 04-uso.md
```

---

## Início rápido

```bash
# 1. Clonar o repositório
git clone https://github.com/C1ceroAnd/rsl_triagem.git
cd rsl_triagem

# 2. Subir o n8n
docker compose up -d

# 3. Iniciar o Ollama (em outro terminal)
ollama serve

# 4. Baixar o modelo (apenas na primeira vez)
ollama pull llama3.2:3b
```

Depois:
1. Acesse **http://localhost:5678** e importe `workflows/FLUXO_TCC_OLLAMA.json`
2. Configure as credenciais Ollama e Google Sheets — veja `docs/02` e `docs/03`
3. Abra `painel/index.html` no navegador e configure as URLs dos webhooks
4. Siga o guia completo em `docs/04-uso.md`

---

## Documentação

| Documento | Conteúdo |
|---|---|
| [01 — Docker](docs/01-configuracao-docker.md) | Subir o n8n, importar workflow, comandos úteis |
| [02 — Ollama](docs/02-configuracao-ollama.md) | Instalar, baixar modelo, credencial no n8n |
| [03 — Google Sheets](docs/03-configuracao-google-sheets.md) | Criar planilha, OAuth2, vincular ao workflow |
| [04 — Uso](docs/04-uso.md) | Fluxo completo de triagem, checklist, resolução de problemas |

---

## Referência de validação

> Rajapakse, R. N., Zahedi, M., & Babar, M. A. (2021). Challenges and solutions when adopting DevSecOps: A systematic review. *Information and Software Technology*, 137, 106700. https://doi.org/10.1016/j.infsof.2021.106700

---

## Autor

Cícero Andrade Santos  
IFPI — Instituto Federal do Piauí, Campus Piripiri  
Orientador: Prof. Me. Jeferson do Nascimento Soares  
TCC 2026

---

## Licença

Uso acadêmico. Para reutilização ou adaptação, cite a fonte.
