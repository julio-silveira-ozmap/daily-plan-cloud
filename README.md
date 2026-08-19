# daily-plan-cloud

Versão **não-interativa** do `daily-plan` (a skill pessoal em
`~/.claude/skills/daily-plan`), feita pra rodar como rotina cloud agendada
(via `/schedule`) todo dia às 8h, sem depender da máquina local estar ligada.

## Por que este repo existe

A rotina cloud roda isolada — sem acesso a `~/.claude/`, aos MCPs locais
(Atlassian HTTP localhost, google-calendar via npx/stdio no WSL) nem ao `gh`
autenticado localmente. Pra funcionar de verdade, ela precisa:

1. De uma skill **de projeto** (`.claude/skills/daily-plan-digest/`,
   carregada pelo Claude Code automaticamente ao clonar este repo) — já que
   o agente cloud começa sem contexto nenhum e não enxerga
   `~/.claude/skills`. É a mesma convenção de skill de sempre, só que
   versionada aqui em vez de pessoal, e adaptada pro modo sem usuário
   presente (ver seção abaixo).
2. De conectores **remotos** de Jira, Google Calendar e GitHub (configurados
   em claude.ai/customize/connectors), não os locais.
3. De um jeito de **persistir progresso entre execuções** sem `~/.claude/` —
   aqui isso é `state/`, versionado neste repo (a rotina faz commit + push a
   cada execução).

## Diferença importante em relação à skill local

A skill local `daily-plan` **dispara automaticamente** as sub-skills
especializadas (`pr-review`, `task-test-prep`, etc.) porque roda com você
presente — cada sub-skill tem seu próprio gate de aprovação antes de
publicar/escrever algo.

A rotina cloud roda **sem ninguém presente** pra aprovar nada. Por isso,
`daily-plan-digest` só produz o **resumo/digest** (leitura, sem disparar
sub-skill nenhuma) — a ação de fato (revisar PR, testar issue, etc.) fica
pra quando você abrir o Claude Code e rodar a skill local `daily-plan`, já
com o resumo de hoje à mão em `state/daily/<hoje>.md`.

## Estrutura

```
.claude/skills/daily-plan-digest/SKILL.md  # skill de projeto, carregada automaticamente
state/STATE.md                              # curado, lido no início de toda execução
state/daily/YYYY-MM-DD.md                   # log bruto do dia, append-only
state/archive/YYYY-MM.md                    # rollup mensal (mesma regra da versão local)
```

Mesmas regras de manutenção de `~/.claude/daily-plan-state/README.md`
(promoção seletiva, poda de dailies com +30 dias).
