# Triagem Automática da Fila de Arrecadação

Projeto de automação para triagem avançada de chamados de suporte Nível 2/3 nas verticais de **Arrecadação**, **Atendimento** e **ISS** da Betha Sistemas.

## Objetivo

Analisar diariamente os chamados recém-abertos no Jira, buscar soluções já validadas na base de conhecimento (chamados históricos) e municiar a equipe de atendimento com sugestões de resolução através de comentários internos.

## Produtos abrangidos

- Tributos
- Procuradoria
- Gestão Fiscal
- e-Nota
- Cidadão Web
- Livro Eletrônico
- Protocolo

## Como funciona

O processo, executado diariamente, é dividido em quatro passos:

1. **Coleta** — listagem dos chamados da fila de triagem via JQL.
2. **Filtro de Idempotência** — ignora chamados já analisados (identificados pela tag `[#IA-TRIAGEM-AUTOMATICA#]`).
3. **Análise e Busca** — pesquisa por soluções validadas em chamados históricos nos projetos `jira-atendimento` e `jira-desenv`.
4. **Registro** — adiciona comentário **interno** com as sugestões encontradas.

> As regras detalhadas, a query JQL completa e o formato do comentário interno estão documentados em [`CLAUDE.md`](./CLAUDE.md).

## Regras críticas

- **Nunca** envia mensagens públicas ao cliente — apenas comentários internos.
- **Nunca** inventa soluções — todas devem vir de chamados históricos confirmados/aprovados.
- Leis e regras de negócio podem ser buscadas externamente, mas sempre identificadas como tal na seção "Análise Complementar".

## Repositório

https://github.com/arimanoelgomes-ctrl/triagem_fila_arrecadacao
