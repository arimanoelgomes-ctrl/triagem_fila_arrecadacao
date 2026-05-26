# Scripts auxiliares

## `post_comentarios.js`

Workaround para postar os comentários da triagem como **Nota Interna** no Jira da Betha enquanto o bug no MCP `@betha/jira-mcp` (documentado em [`docs/incidente_mcp_add_comment.md`](../docs/incidente_mcp_add_comment.md)) não é corrigido.

O script chama diretamente a API REST do Jira (`POST /rest/api/2/issue/{key}/comment`) com o payload correto:

```json
{
    "body": "...",
    "properties": [
        { "key": "sd.public.comment", "value": { "internal": true } }
    ]
}
```

### Requisitos

- **Node.js 18 ou superior** (usa `fetch` nativo — sem dependências externas).
- Credenciais do Jira (mesmas usadas pelo MCP, ou suas pessoais).

Verifique a versão do Node:

```cmd
node --version
```

Se for inferior a 18, atualize antes de continuar.

### Setup (primeira vez)

1. Entre no diretório:

   ```cmd
   cd C:\Scripts\ias\projetos_de_ia\triagem_fila_arrecadacao\scripts
   ```

2. Crie o `.env` a partir do template:

   ```cmd
   copy .env.example .env
   ```

3. Abra `scripts\.env` num editor e preencha:

   ```
   JIRA_BASE_URL=https://atendimento.betha.com.br
   JIRA_USERNAME=seu_usuario
   JIRA_PASSWORD=sua_senha_ou_token
   ```

   > **Importante:** `scripts/.env` está no `.gitignore` do projeto — nunca será commitado.

### Uso

#### Listar arquivos de comentários disponíveis

```cmd
node post_comentarios.js --list
```

Mostra os `outputs/*_comentarios_para_postar.md` existentes.

#### Dry-run (não posta — apenas mostra o que seria postado)

```cmd
node post_comentarios.js --dry-run
```

Por padrão pega o arquivo mais recente em `outputs/`. Para apontar para um arquivo específico:

```cmd
node post_comentarios.js --dry-run --file ../outputs/2026-05-25_comentarios_para_postar.md
```

#### Postar de verdade

```cmd
node post_comentarios.js
```

O script vai:

1. Listar os comentários candidatos.
2. **Pedir confirmação interativa** (`s/N`).
3. Para cada comentário:
   - Verificar idempotência (se já existe comentário com `[#IA-TRIAGEM-AUTOMATICA#]` no chamado, **ignora**).
   - Postar como **Nota Interna**.
4. Mostrar resumo final com totais (postados / ignorados / erros).

Para pular a confirmação interativa (uso em scripts):

```cmd
node post_comentarios.js --yes
```

### Saída esperada

```
📄 Arquivo:  C:\...\outputs\2026-05-25_comentarios_para_postar.md
🌐 Jira:     https://atendimento.betha.com.br
🧪 Modo:     POSTAGEM REAL

Encontrados 4 comentário(s) candidato(s):
  1. BTHSC-319007  →  [#IA-TRIAGEM-AUTOMATICA#] | 🤖 **Triagem Automática de Soluções**...
  2. BTHSC-312959  →  ...
  3. BTHSC-318276  →  ...
  4. BTHSC-317104  →  ...

Confirma postagem dos 4 comentários como NOTA INTERNA? [s/N]: s
• BTHSC-319007: OK — comentário 14237750 postado como INTERNO.
• BTHSC-312959: OK — comentário 14237751 postado como INTERNO.
• BTHSC-318276: OK — comentário 14237752 postado como INTERNO.
• BTHSC-317104: OK — comentário 14237753 postado como INTERNO.

─────────────────────────────────────────────
Postados: 4   Ignorados: 0   Erros: 0
─────────────────────────────────────────────
```

### Validação pós-postagem

**Sempre** confirme visualmente no Jira que pelo menos um dos comentários ficou com o destaque amarelado de Nota Interna (classe CSS `js-sd-internal-comment active`). Se algo der errado, os comentários podem ser removidos via API ou interface.

### Quando descartar este script

Após o fix oficial do MCP `@betha/jira-mcp` ser entregue e validado (ver critério de aceitação em [`docs/incidente_mcp_add_comment.md`](../docs/incidente_mcp_add_comment.md)), este script pode ser arquivado ou removido — a triagem automatizada voltará a postar diretamente via MCP.
