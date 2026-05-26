# Incidente — MCP `jira-atendimento__add_comment` não respeita `properties`

**Data:** 2026-05-25
**Severidade:** Alta (risco de exposição de comentário interno como público)
**Status:** Aberto — aguardando ajuste no MCP

## Identificação do componente

- **Pacote NPM:** `@betha/jira-mcp`
- **Registry:** `http://nexus3.betha.com.br/repository/npm-all/`
- **Forma de execução:** `npx -y --registry http://nexus3.betha.com.br/repository/npm-all/ @betha/jira-mcp`
- **Ferramenta afetada:** `add_comment` (assume `comment`, `explicitUserRequest`, e aparentemente aceita `properties` no schema mas não o repassa)

---

## Resumo

O MCP interno `jira-atendimento__add_comment` aceita o parâmetro `properties` no schema (não retorna erro de validação), mas **não o repassa para a API REST do Jira**. Como consequência, comentários que deveriam ser marcados como **Nota Interna** (`sd.public.comment` com `internal: true`) acabam sendo gravados como **comentários públicos** — visíveis ao cliente.

Este incidente foi identificado durante a primeira execução de validação do projeto **Triagem Automática da Fila de Arrecadação**, onde a regra crítica de segurança proíbe postagem pública.

---

## Reprodução

### 1. Payload esperado pela API do Jira (capturado via DevTools)

Endpoint: `POST https://atendimento.betha.com.br/rest/api/2/issue/{issueId}/comment`

```json
{
    "body": "teste interno",
    "properties": [
        {
            "key": "sd.public.comment",
            "value": {
                "internal": true
            }
        }
    ]
}
```

Quando enviado pela interface do Jira (botão de Nota Interna ativo, com classe CSS `js-sd-internal-comment active`), o comentário é gravado como **interno**.

### 2. Chamada via MCP equivalente

```
mcp__jira-atendimento__add_comment(
  issueKey: "BTHSC-318167",
  comment: "teste interno via MCP",
  explicitUserRequest: true,
  properties: [{"key": "sd.public.comment", "value": {"internal": true}}]
)
```

**Resultado:**
- O MCP retornou sucesso (`commentId: 14237696`).
- Porém, ao inspecionar visualmente no Jira, o comentário ficou **público** (sem o destaque amarelado de nota interna).
- O comentário foi imediatamente removido via `mcp__jira-atendimento__delete_comment`.

### 3. Variações já testadas (todas falharam)

| Tentativa | Parâmetros | Resultado |
|-----------|------------|-----------|
| A | `body` + `properties` | Erro 400 — schema rejeitou `body` (esperava `comment`) |
| B | `comment` + `explicitUserRequest: true` + `internal: true` + `properties` | Erro 400 |
| C | `comment` + `explicitUserRequest: true` + `properties` | Sucesso (200), mas **comentário ficou público** |

---

## Diagnóstico

O parâmetro `properties` está documentado no schema do MCP (aceito sem erro), mas o wrapper **não o serializa no body do request HTTP** enviado ao Jira. Hipóteses:

1. **Bug de propagação:** o handler do `add_comment` ignora silenciosamente o campo `properties` antes de chamar a API.
2. **Conversão indevida:** o campo é convertido para uma estrutura que o Jira não reconhece (e a API silenciosamente descarta).
3. **Flag alternativa não exposta:** o MCP pode ter uma flag interna (`internal`, `isInternal`, `commentType`, `noteType`) que ainda não foi exposta no schema público — e por isso a única forma seria essa.

---

## Solicitação ao time responsável pelo MCP

Pedimos uma das duas correções (em ordem de preferência):

1. **Repassar `properties` corretamente:** se o consumidor enviar `properties: [{key, value}]`, o MCP deve serializar esse array no body do POST `/rest/api/2/issue/{id}/comment` exatamente como recebido.

2. **OU expor flag de conveniência `internal: true`:** o MCP recebe `internal: true` e monta internamente o `properties: [{key: "sd.public.comment", value: {internal: true}}]` antes de chamar a API.

A opção 1 é mais geral (cobre outras properties no futuro); a opção 2 é mais idiomática para o caso de uso comum.

---

## Impacto operacional enquanto o MCP não é corrigido

- Todos os comentários da triagem automatizada são **gerados em arquivo Markdown** (`outputs/YYYY-MM-DD_comentarios_para_postar.md`) e colados manualmente no Jira pelo coordenador.
- O log diário (`logs/YYYY-MM-DD.md`) registra os comentários como "preparados, não postados".
- A postagem automática **fica suspensa** até o ajuste do MCP ser entregue e validado.

---

## Como validar o fix

Após o ajuste, executar este teste em um chamado controlado (ex.: novo chamado de teste, ou um já fechado):

```
mcp__jira-atendimento__add_comment(
  issueKey: "<chamado-teste>",
  comment: "teste de nota interna pós-fix",
  explicitUserRequest: true,
  properties: [{"key": "sd.public.comment", "value": {"internal": true}}]
)
```

**Critério de aceitação:** o comentário aparece no Jira com a marcação visual de nota interna (fundo amarelado, `js-sd-internal-comment active`) e **não dispara notificação ao cliente** no e-mail do reporter.

---

## Histórico de ações tomadas em 2026-05-25

- 17:00 — Triagem identificou 34 chamados na fila.
- ~18:00 — 4 comentários preparados em `outputs/2026-05-25_comentarios_para_postar.md`.
- ~18:30 — Tentativa de postagem via MCP no BTHSC-318167 com `properties`. Comentário 14237696 criado mas ficou público.
- ~18:32 — Comentário 14237696 removido via `delete_comment`. Sem prejuízo ao cliente (intervalo curto, conteúdo do teste era apenas a string "teste interno via MCP").
- Documento `docs/incidente_mcp_add_comment.md` criado.
- Log `logs/2026-05-25.md` atualizado.
