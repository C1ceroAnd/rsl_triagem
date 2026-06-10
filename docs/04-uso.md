# 04 — Uso do sistema

Com o ambiente configurado (Docker, Ollama e Google Sheets), este documento descreve o fluxo completo de uso.

---

## Pré-execução — checklist

Antes de iniciar qualquer triagem, confirme:

- [ ] Docker Desktop está em execução
- [ ] Container n8n está rodando: `docker ps` deve listar `n8n_tcc`
- [ ] Ollama está servindo: `curl http://localhost:11434/api/tags` retorna os modelos
- [ ] Workflow está ativo no n8n (botão de toggle no canto superior direito do workflow)
- [ ] Planilha Google Sheets está acessível pela conta configurada

---

## Abrindo o painel

Abra o arquivo `painel/index.html` diretamente no navegador:

```bash
open painel/index.html
```

Ou arraste o arquivo para uma janela do navegador.

Na aba **Configurações**, confirme que as URLs estão preenchidas:

- **URL do Webhook — Triagem:** `http://localhost:5678/webhook/Triagem_RSL`
- **URL do Webhook — Resultados:** `http://localhost:5678/webhook/Resultados-rsl`

Clique em **Salvar** — as URLs ficam armazenadas no `localStorage` do navegador.

---

## Exportando arquivos das bases de dados

O sistema aceita arquivos `.bib` (BibTeX) e `.ris` exportados diretamente das bases:

| Base | Formato recomendado | Observação |
|---|---|---|
| IEEE Xplore | BibTeX (.bib) | Export direto na página de resultados |
| Scopus | BibTeX (.bib) | Selecionar todos → Export → BibTeX |
| ACM Digital Library | BibTeX (.bib) | Usar `dl.acm.org` diretamente — exportações via portal CAPES podem conter HTML |

---

## Executando a triagem

1. Abra o painel (`painel/index.html`)
2. Na aba **Critérios**, defina os critérios de inclusão e exclusão da sua RSL e clique em **Salvar Critérios**
3. Na aba **Upload & Triagem**, selecione os arquivos `.bib` ou `.ris`
4. Clique em **Executar Triagem**

O painel envia todos os arquivos em uma única requisição ao n8n. O workflow processa cada artigo individualmente — o tempo total depende da quantidade de registros e da velocidade do modelo no hardware local.

---

## O que acontece no pipeline

```
Painel envia JSON com array de arquivos + critérios
    │
    ▼
[Listar Arquivos]     extrai conteúdo binário de cada arquivo
    │
    ▼
[Parser BibTeX/RIS]   detecta base de origem, divide entradas, converte para JSON
    │
    ▼
[Limpeza]             normaliza campos: titulo, autores, ano, doi, resumo, fonte
    │
    ▼
[AI Agent]            envia título + resumo + critérios para llama3.2:3b
    │                 retorna: decisao / confianca / justificativa
    ▼
[Organizador]         aplica exclusão determinística por data (JS, não LLM)
    │                 parseia resposta JSON do agente
    ▼
[Google Sheets]       grava uma linha por artigo na planilha
```

> A verificação de intervalo de datas é feita em JavaScript no nó "Organizador", não pelo modelo. Artigos fora do período recebem `EXCLUIR` com `confianca: 99` sem passar pelo LLM — isso elimina falsos negativos causados por variação de comportamento do modelo nessa checagem.

---

## Consultando os resultados

Na aba **Resultados** do painel, clique em **Atualizar**. O painel consulta o segundo webhook (`Resultados-rsl`) que lê as linhas da planilha e retorna os dados para exibição.

Use o filtro por decisão para visualizar apenas `INCLUIR`, `EXCLUIR` ou `REVISAR`.

---

## Validação humana

A coluna `validacao_humana` na planilha é reservada para revisão manual. Para artigos com decisão `REVISAR` ou com confiança abaixo do limiar aceitável, o pesquisador pode preencher essa coluna diretamente na planilha com `CONFIRMAR` ou `REJEITAR`.

---

## Resolução de problemas comuns

**O painel não consegue conectar ao webhook**
- Confirme que o n8n está rodando: `docker ps`
- Confirme que o workflow está ativo (toggle no n8n)
- Verifique se a URL do webhook no painel está correta: `http://localhost:5678/webhook/Triagem_RSL`

**O Ollama não responde**
- Execute `ollama serve` no terminal
- Confirme que o modelo está disponível: `ollama list`

**Artigos sem resumo recebem sempre REVISAR**
- Comportamento esperado — sem abstract, o modelo não tem informação suficiente para decidir

**Token do Google Sheets expirou**
- Acesse Credentials no n8n, edite a credencial Google Sheets e reautentique
