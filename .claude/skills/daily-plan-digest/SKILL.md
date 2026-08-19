---
name: daily-plan-digest
description: 'Use quando: rodando como rotina cloud agendada (sem o usuário presente) — coleta Jira + GitHub + Calendar via MCP connectors, monta o resumo do dia e persiste em state/, sem disparar nenhuma ação que precise de aprovação.'
argument-hint: '[vazio — sempre roda o digest completo]'
---

# daily-plan-digest

Versão **cloud, não-interativa** da skill pessoal `daily-plan`
(`~/.claude/skills/daily-plan`, mantida à parte no ambiente local do
usuário). Roda sem ninguém presente — por isso só produz **resumo/registro**,
nunca dispara sub-skill nem publica nada que exija aprovação.

## Use quando

- Sessão cloud agendada (rotina `/schedule`) sem usuário presente.

## Não use quando

- Há um usuário na conversa que pode aprovar ações — nesse caso use a skill
  local `daily-plan`, que já dispara as sub-skills especializadas.

## Bloqueios

- Comentar, transicionar, aprovar/reprovar PR, ou qualquer escrita em
  Jira/GitHub — só leitura.
- Disparar `pr-review`, `resolving-pr-feedback`, `task-test-prep`,
  `support-investigation` ou `call-prep` — essas skills têm gates de
  aprovação que ninguém vai responder numa execução agendada.
- Fingir que uma fonte respondeu quando o MCP/connector correspondente não
  estava disponível na sessão — declarar a ausência explicitamente.
- Reescrever `state/daily/<hoje>.md` inteiro — é sempre append.

## Passo a passo

### 1. Descobrir quais fontes estão disponíveis nesta sessão

Antes de qualquer coisa, liste as tools MCP conectadas a esta sessão (Jira/
Atlassian, Google Calendar, GitHub). O que não estiver disponível vira nota
explícita no resumo, não suposição silenciosa.

### 2. Sincronizar o state antes de coletar

O progresso real evolui na branch `claude/daily-plan-state`, não na `main`
(rotinas cloud só têm permissão de push em branches com prefixo `claude/` —
ver seção "Persistência" no README deste repo). Antes de ler:

```bash
git fetch origin claude/daily-plan-state 2>/dev/null && \
  git checkout origin/claude/daily-plan-state -- state/
```

Se esse comando falhar (branch ainda não existe — primeira execução), seguir
com o `state/` da `main` como ponto de partida — ele já vira o estado inicial
na primeira vez que a branch `claude/daily-plan-state` for criada no passo 6.
Ler `state/STATE.md`: decisões vigentes e bloqueios ativos do dia anterior
entram no plano de hoje sem redescobrir.

### 3. Coletar dados

- **Jira**: `assignee = currentUser() AND statusCategory != Done ORDER BY
  updated DESC`, cross-project. Status, due date, prioridade, labels,
  sprint, projeto.
- **GitHub**: PRs com review pedido a mim (`is:open review-requested:@me`),
  meus PRs abertos (`is:open author:@me`), PRs onde fui mencionado
  (`is:open mentions:@me`). Se o acesso GitHub estiver limitado a repo(s)
  específico(s) desta sessão, declarar isso — não apresentar como cobertura
  total.
- **Calendar**: eventos de hoje, se o connector responder.

### 4. Calcular sinais objetivos

Aging, due date, status de bloqueio, tamanho do PR, sprint days remaining,
papel no item (assigned/review-requested/mentioned/authored) — calculado a
partir do dado bruto, nunca estimado.

### 5. Montar o resumo em seções fixas

1. Bloqueadores / SLA estourado
2. Reviews pedidas a mim (aging)
3. Meus PRs aguardando ação
4. Itens do sprint com due date hoje/amanhã
5. Calls do dia (se o connector de Calendar respondeu)
6. Quick wins opcionais (1-2 itens)

Cada linha com rationale curto. Fechar com: "Rode `daily-plan` localmente
pra agir sobre estes itens."

### 6. Registrar e persistir

1. Append em `state/daily/<hoje, YYYY-MM-DD>.md` — nunca reescrever o
   arquivo inteiro.
2. Atualizar `state/STATE.md` só com decisões/bloqueios que ainda valem.
3. Se `state/daily/` tiver arquivos com mais de ~30 dias sem consolidar,
   gerar `state/archive/YYYY-MM.md` e remover os dailies daquele mês.
4. Commitar e dar push **na branch `claude/daily-plan-state`**, nunca em
   `main`:
   ```bash
   git add state/ && \
   git commit -m "daily-plan: resumo de <hoje>" && \
   git push origin HEAD:claude/daily-plan-state
   ```
   Isso cria a branch remotamente na primeira execução e só a atualiza nas
   seguintes — sem tocar em `main`. Reportar erro claramente se o push
   falhar (não tentar contornar sozinha, ex. forçando push em `main`).
