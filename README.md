# RSL Triagem — Ferramenta de Triagem Automatizada para Revisões Sistemáticas da Literatura

> Ferramenta desenvolvida como Trabalho de Conclusão de Curso (TCC) no IFPI — Campus Piripiri (2025).  
> Automatiza a triagem de artigos científicos em processos de RSL utilizando Inteligência Artificial.

---

## Visão Geral

O **RSL Triagem** é um sistema que automatiza a etapa de triagem em Revisões Sistemáticas da Literatura. O pesquisador define os critérios de inclusão e exclusão, faz o upload dos arquivos bibliográficos exportados das bases de dados, e a IA processa cada artigo retornando uma decisão fundamentada.

O sistema é agnóstico de área — funciona para qualquer tema de pesquisa.

**Fluxo geral:**

```
Pesquisador define critérios
        ↓
Upload de arquivos .bib ou .ris
        ↓
n8n processa via Agente de IA (Groq)
        ↓
Resultados salvos no Google Sheets
        ↓
Painel exibe decisões, justificativas e confiança
```

---

## Funcionalidades

- Upload de arquivos `.bib` (BibTeX) e `.ris` diretamente no painel
- Triagem automatizada com IA (decisão: **INCLUIR**, **EXCLUIR** ou **REVISAR**)
- Justificativa e índice de confiança (0–100%) para cada decisão
- Critérios de inclusão e exclusão configuráveis pelo pesquisador
- Resultados persistidos no Google Sheets
- Visualização dos resultados com filtro por decisão
- Painel 100% web — sem instalação necessária para o pesquisador

---

## Tecnologias Utilizadas

| Componente | Tecnologia |
|---|---|
| Interface | HTML + CSS + JavaScript (single page) |
| Hospedagem do painel | GitHub Pages |
| Automação e orquestração | n8n (Railway) |
| Agente de IA | Groq (LLM) |
| Armazenamento de resultados | Google Sheets |

---

## Pré-requisitos para Replicação

Para replicar este projeto, você precisará de:

- Conta no [n8n Cloud](https://n8n.io) ou instância self-hosted (Railway recomendado)
- Conta no [Groq](https://console.groq.com) para obter a chave de API
- Conta Google com acesso ao Google Sheets
- Conta no GitHub para hospedar o painel via GitHub Pages

---

## Estrutura do Repositório

```
rsl_triagem/
└── index.html        # Painel completo (interface, estilos e lógica)
└── README.md         # Documentação do projeto
```

---

## Como Usar

### 1. Acesse o Painel

Acesse o painel pelo GitHub Pages:  
👉 [https://c1ceroand.github.io/rsl_triagem/](https://c1ceroand.github.io/rsl_triagem/)

---

### 2. Configure as URLs (aba Configurações)

Antes de usar, configure as URLs do seu ambiente n8n:

| Campo | Descrição |
|---|---|
| URL do Webhook — Triagem | URL de produção do workflow de triagem no n8n |
| URL do Webhook — Resultados | URL de produção do workflow de resultados no n8n |
| URL da Planilha Google Sheets | URL pública (pubhtml) da planilha de resultados |

> As configurações são salvas localmente no navegador.

---

### 3. Defina os Critérios (aba Critérios)

Na aba **Critérios**, defina:

- **Critérios de Inclusão** — o artigo será incluído se atender a pelo menos um
- **Critérios de Exclusão** — o artigo será excluído se atender a pelo menos um

Exemplo:
```
Inclusão:
- Artigo revisado por pares
- Publicado entre 2019 e 2025
- Aborda aprendizado de máquina aplicado à saúde

Exclusão:
- Artigo duplicado
- Fora do idioma inglês ou português
- Resumo indisponível
```

---

### 4. Faça o Upload dos Arquivos (aba Upload & Triagem)

1. Exporte os artigos das bases de dados no formato `.bib` ou `.ris`
2. Arraste os arquivos para a área de upload ou clique para selecionar
3. Clique em **Executar Triagem**
4. Aguarde o processamento — cada arquivo é enviado individualmente ao n8n

---

### 5. Visualize os Resultados (aba Resultados)

Clique em **Atualizar** para carregar os resultados processados. O painel exibe:

- Total de artigos processados
- Contagem por decisão (Incluir / Excluir / Revisar)
- Tabela com título, autores, ano, fonte, decisão e confiança da IA
- Filtro por tipo de decisão

---

### 6. Planilha Google Sheets

A aba **Planilha Google** exibe os dados diretamente da planilha em tempo real. Você também pode abrir a planilha completa para edição manual, validação humana ou exportação.

---

## Configuração do n8n

O sistema utiliza dois workflows no n8n:

### Workflow 1 — Triagem RSL

Responsável por receber os arquivos, processar com a IA e salvar no Sheets.

```
Webhook (POST /Triagem_RSL)
    → Código JS (parser BibTeX/RIS → JSON)
    → Agente de IA (Groq)
    → Google Sheets (salvar resultado)
    → Respond to Webhook
```

**Prompt do Agente de IA:**

```
Você é um assistente especializado em triagem de artigos científicos para 
Revisões Sistemáticas da Literatura (RSL).

Sua tarefa é analisar o título e o resumo de um artigo e decidir se ele deve 
ser INCLUÍDO, EXCLUÍDO ou marcado para REVISÃO HUMANA, com base exclusivamente 
nos critérios fornecidos pelo pesquisador.

CRITÉRIOS DE INCLUSÃO (fornecidos pelo pesquisador):
{{criterios_inclusao}}

CRITÉRIOS DE EXCLUSÃO (fornecidos pelo pesquisador):
{{criterios_exclusao}}

REGRAS DE DECISÃO:
- INCLUIR: o artigo atende a pelo menos UM critério de inclusão e nenhum de exclusão
- EXCLUIR: o artigo atende a pelo menos UM critério de exclusão
- REVISAR: há ambiguidade, informações insuficientes ou conflito entre critérios

IMPORTANTE:
- Em caso de dúvida, prefira REVISAR a EXCLUIR
- Baseie a análise apenas no título e resumo fornecidos
- Seja consistente independentemente do tema da pesquisa

Responda APENAS no seguinte formato JSON:
{
  "decisao": "INCLUIR" ou "EXCLUIR" ou "REVISAR",
  "confianca": número de 0 a 100,
  "justificativa": "explicação breve em português"
}
```

---

### Workflow 2 — Resultados RSL

Responsável por retornar os dados do Sheets para o painel.

```
Webhook (POST /Resultados-rsl)
    → Google Sheets (Get Rows)
    → Código JS (agregar itens)
    → Respond to Webhook (All Incoming Items)
```

**Configuração do Webhook:**
- Respond: `Using 'Respond to Webhook' Node`

**Código JS do nó de agregação:**
```javascript
const todos = $input.all().map(i => i.json);
return [{ json: { dados: todos } }];
```

**Respond to Webhook:**
- Respond With: `All Incoming Items`

---

## Formato dos Arquivos de Entrada

O sistema aceita arquivos exportados diretamente das principais bases de dados:

**BibTeX (.bib)** — exportado do Scopus, Web of Science, Google Scholar:
```bibtex
@article{silva2023,
  title     = {Título do artigo},
  author    = {Silva, J. and Santos, M.},
  journal   = {Journal Name},
  year      = {2023},
  doi       = {10.xxxx/xxxxx},
  abstract  = {Resumo do artigo...}
}
```

**RIS (.ris)** — exportado do Mendeley, Zotero, PubMed:
```
TY  - JOUR
TI  - Título do artigo
AU  - Silva, J.
PY  - 2023
DO  - 10.xxxx/xxxxx
AB  - Resumo do artigo...
ER  -
```

---

## Limitações Conhecidas

- A qualidade da triagem depende da presença do **resumo (abstract)** no arquivo exportado — artigos sem resumo tendem a receber decisão **REVISAR**
- O sistema processa um arquivo por vez — lotes muito grandes podem demorar dependendo do plano do n8n e Groq
- As configurações de URL são salvas no `localStorage` do navegador — ao trocar de dispositivo será necessário reconfigurar

---

## Autor

Desenvolvido por **Cícero Andrade Santos**  
IFPI — Instituto Federal do Piauí, Campus Piripiri  
TCC 2026

---

## Licença

Este projeto é de uso acadêmico. Para reutilização ou adaptação, cite a fonte.
