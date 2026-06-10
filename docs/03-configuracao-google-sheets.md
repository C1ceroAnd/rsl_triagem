# 03 — Configuração do Google Sheets

O n8n salva os resultados da triagem em uma planilha do Google Sheets via OAuth2. Esta seção cobre a criação da planilha, a configuração da credencial e a vinculação ao workflow.

---

## 1. Criar a planilha

Acesse **https://sheets.google.com** e crie uma nova planilha.

Na primeira linha (cabeçalho), crie as seguintes colunas **exatamente nessa ordem e com esses nomes**:

| A | B | C | D | E | F | G | H | I |
|---|---|---|---|---|---|---|---|---|
| titulo | autores | ano | doi | fonte | decisao | confianca | justificativa | validacao_humana |

> O n8n usa `autoMapInputData` para mapear os campos — os nomes das colunas precisam corresponder exatamente aos campos retornados pelo nó "Organizador de resultados".

Anote o **ID da planilha**: é a sequência de caracteres na URL entre `/d/` e `/edit`.

```
https://docs.google.com/spreadsheets/d/ESTE_TRECHO_E_O_ID/edit
```

---

## 2. Configurar a credencial OAuth2 no n8n

1. Acesse **http://localhost:5678**
2. No menu lateral, vá em **Credentials**
3. Clique em **Add credential**
4. Busque por **Google Sheets OAuth2 API** e selecione
5. Clique em **Sign in with Google**
6. Uma janela de autenticação do Google abrirá — faça login com a conta que tem acesso à planilha
7. Autorize as permissões solicitadas pelo n8n
8. Clique em **Save**

> O n8n armazena o token de acesso de forma criptografada no volume `n8n_data`. O token é renovado automaticamente enquanto o container estiver ativo.

---

## 3. Vincular a credencial ao workflow

1. Abra o workflow importado
2. Clique no nó **Append row in sheet**
3. No campo **Credential**, selecione a credencial Google Sheets recém-criada
4. No campo **Document**, selecione a planilha criada no passo 1
5. No campo **Sheet**, selecione a aba (por padrão: `Página1`)
6. Salve o workflow

---

## 4. Verificar o mapeamento de colunas

No nó **Append row in sheet**, confirme que o mapeamento está configurado como **Auto-map input data**. Com as colunas nomeadas corretamente, o n8n mapeia os campos automaticamente sem configuração manual.

---

## Sobre a expiração do token

Tokens OAuth2 com contas Google pessoais têm validade longa e são renovados automaticamente pelo n8n em uso normal. Se o token expirar (por inatividade prolongada ou mudança de senha), repita o passo 2 para reautenticar.

Se estiver usando uma conta institucional (Google Workspace), o token pode expirar por política da instituição. Nesse caso, use uma conta Google pessoal ou configure um projeto GCP próprio com o app OAuth publicado.
