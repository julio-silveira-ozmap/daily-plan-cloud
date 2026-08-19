# RUN — daily-plan (versão cloud, digest-only)

Você está rodando sem o usuário presente (rotina agendada). Sua saída é
**só leitura + registro** — nunca dispare ação que precise de aprovação
(publicar comentário, abrir/editar PR, transicionar issue). Ninguém vai
responder a um pedido de confirmação nesta execução.

## 1. Ler o state antes de tudo

Leia `state/STATE.md` neste repo (já clonado) — decisões vigentes e
bloqueios ativos do dia anterior entram no plano de hoje sem redescobrir.

## 2. Coletar dados (via MCP connectors conectados a esta rotina)

- **Jira**: busque issues com `assignee = currentUser() AND statusCategory
  != Done ORDER BY updated DESC`, cross-project (cobre todos os boards
  automaticamente). Traga status, due date, prioridade, labels, sprint,
  projeto.
- **GitHub**: PRs com review pedido pra mim (`is:open review-requested:@me`),
  meus PRs abertos (`is:open author:@me`), PRs onde fui mencionado
  (`is:open mentions:@me`). Se só houver acesso a um repo específico via
  checkout, deixe explícito no resumo que a busca de PR está limitada a esse
  repo, não a todos.
- **Calendar**: eventos de hoje. Se o connector de Calendar não estiver
  disponível nesta execução, omita a seção 5 do resumo e avise isso.

Se alguma fonte não estiver acessível (connector não conectado), **não
finja que rodou** — declare explicitamente qual fonte faltou.

## 3. Calcular sinais objetivos

Aging (horas desde criação/review pedida), due date (dias até prazo), status
de bloqueio, tamanho do PR, sprint days remaining, papel no item
(assigned/review-requested/mentioned/authored). Sem estimativa do modelo —
calcule a partir dos dados brutos.

## 4. Montar o resumo em seções fixas

1. Bloqueadores / SLA estourado
2. Reviews pedidas a mim (aging)
3. Meus PRs aguardando ação
4. Itens do sprint com due date hoje/amanhã
5. Calls do dia (se o connector de Calendar respondeu)
6. Quick wins opcionais (1-2 itens pequenos)

Cada linha com rationale curto (por que está ali). **Não dispare nenhuma
sub-skill/ação** — isso é só o digest. Termine com uma nota:
"Rode `daily-plan` localmente pra agir sobre estes itens."

## 5. Registrar e persistir

1. Append em `state/daily/<hoje, formato YYYY-MM-DD>.md`: o resumo completo
   gerado hoje (nunca sobrescreva o arquivo do dia, só acrescente se já
   existir uma entrada de execução anterior no mesmo dia).
2. Atualize `state/STATE.md` só com decisões/bloqueios que ainda valem pro
   próximo dia (não a narrativa toda).
3. Se `state/daily/` tiver arquivos com mais de ~30 dias sem consolidar,
   gere `state/archive/YYYY-MM.md` com o resumo daquele mês e remova os
   dailies individuais correspondentes.
4. `git add -A && git commit -m "daily-plan: resumo de <hoje>" && git push`.
   Se o push falhar (ex. sem permissão), reporte o erro claramente em vez de
   silenciar.
