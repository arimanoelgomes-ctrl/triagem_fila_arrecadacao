# Logs de Execução da Triagem Automática

Esta pasta armazena o histórico diário de execução da triagem. Cada execução gera um arquivo Markdown no formato `YYYY-MM-DD.md`.

## Finalidade

- **Auditoria** — rastrear o que foi analisado, ignorado ou comentado em cada dia.
- **Qualidade** — permitir revisão amostral pelo coordenador de portfólio.
- **Diagnóstico** — investigar comportamentos inesperados da IA (falsos negativos, comentários incorretos etc.).

## Nomenclatura

`logs/YYYY-MM-DD.md` — um arquivo por dia de execução (ex.: `2026-05-25.md`).

## Formato do log

Cada arquivo deve seguir o template abaixo:

```markdown
# Triagem Automática — YYYY-MM-DD

**Início da execução:** HH:MM (BRT)
**Fim da execução:** HH:MM (BRT)
**Total de chamados retornados pela JQL:** N
**Ignorados (já analisados):** N
**Analisados nesta execução:** N
**Comentados (com sugestão):** N
**Sem comentário (sem solução histórica e sem leis aplicáveis):** N

---

## Chamados ignorados (idempotência)

| Chave | Resumo | Motivo |
|-------|--------|--------|
| ATEND-1234 | ... | Já possui tag [#IA-TRIAGEM-AUTOMATICA#] |

## Chamados comentados

### ATEND-5678 — Resumo curto do chamado
- **Vertical/Sistema:** Arrecadação / Tributos
- **Município:** Itajaí
- **Chamados históricos utilizados:** ATEND-1111, ATEND-2222
- **Resumo da sugestão aplicada:** ...

## Chamados analisados sem comentário

| Chave | Resumo | Motivo de não comentar |
|-------|--------|------------------------|
| ATEND-9999 | ... | Nenhum histórico aprovado encontrado |

## Observações / incidentes

- (Erros de API, chamados problemáticos, ajustes a discutir etc.)
```

## Regras

- Os logs são **versionados no Git** para servir de trilha de auditoria.
- Nunca incluir nos logs dados sensíveis do contribuinte (CPF/CNPJ completo, valores específicos, endereços etc.). Quando necessário citar, anonimizar.
- O log deve ser gerado **mesmo quando a fila estiver vazia** ou quando todos os chamados forem ignorados — neste caso, o arquivo registra apenas o cabeçalho com totais zerados.
