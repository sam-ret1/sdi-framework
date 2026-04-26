# Triage questions (Phase 1)

Exactly 5 questions. Each accepts **"não sei / pular"** as a valid answer with a defined skip behavior. Do not ask follow-ups.

The audit ran in Phase 0 and produced a discovery report. These questions confirm or correct the report's findings.

## Asking style

Batch all 5 in one message, OR ask one at a time if the user prefers (some users handle batches better; others prefer step-by-step). Either way, keep the questions short, with the auto-detected default visible so the user can confirm with a single word.

## The 5 questions

### Q1 — Project type

```
**Q1.** O auto-audit detectou tipo: **[Type detected]** (confiança: [high/medium/low]).

Confirma? Se errado, qual dos 8 tipos é correto?
1. Web app SaaS multi-tenant
2. Landing page / marketing site
3. Dashboard / admin / internal tool
4. Web API / backend service
5. Mobile app
6. Data pipeline / scraper + reports
7. AI agent / MCP server / chatbot
8. Integration / automation workflow
9. Outro — descreva
0. Não sei / pular
```

**Skip behavior (0):** Keep the auto-detected type with a low-confidence flag in AGENTS.md. The framework still works; user can correct later by editing AGENTS.md.

### Q2 — AI modifier

```
**Q2.** Auto-audit detectou AI modifier: **[yes/no]** ([signals if yes]).

Tem componente significativo de LLM/IA no produto? sim / não / não sei
```

**Skip behavior (não sei):** Use the auto-detected value. If signals were present, mark as "yes (auto-detected)"; if not, mark as "no (auto-detected)". The user can flip this later.

### Q3 — Project stage

```
**Q3.** Auto-audit sugere estágio: **[stage]** ([N] indicadores de produção detectados).

Em qual estágio está o projeto?
(a) Greenfield / MVP em construção (sem usuários reais)
(b) MVP lançado, primeiros usuários
(c) Produção madura, vários clientes ativos
(d) Manutenção / poucas mudanças, evoluindo lentamente
(e) Não sei / pular
```

**Skip behavior (e):** Use the auto-detected stage. If 3+ production indicators were detected, treat as `mature-production` and emit the `Production constraints` section in AGENTS.md (the indicators justify it even without confirmation). If 0–2 indicators, mark stage as "unknown" and skip the production section — user can add later.

### Q4 — Work cadence

```
**Q4.** Como o trabalho é organizado hoje?
(a) Fases discretas planejadas (Phase 1, 2, 3...)
(b) Features contínuas (sem fases rígidas, vai escopando uma a uma)
(c) Maioria manutenção e fixes (poucas features novas)
(d) Misto
(e) Não sei / pular
```

**Skip behavior (e):** Mark as `mixed`. The framework supports both — naming will be decided per work item in Phase 3 and subsequent Phase E runs.

### Q5 — Anything material I missed

```
**Q5.** Tem algo crítico do projeto que esse audit não pegaria? Por exemplo:
- Clientes específicos com contratos / SLAs
- Restrições de compliance (GDPR, LGPD, HIPAA, SOC2)
- Deadlines comerciais / regulatórios
- Estrutura de on-call / runbook em ferramenta externa
- Decisões arquitetônicas tomadas off-repo (em Slack, Notion, etc.)
- Incidentes recentes que ainda informam decisões

Qualquer coisa relevante? (responda livre, ou "nada" / "não sei")
```

**Skip behavior ("nada" or "não sei"):** Leave the `Critical context` section in AGENTS.md as a placeholder with a `> Source: best-effort placeholder — populate as you learn the project` flag. User adds context as it surfaces.

## After Q5

Confirm understanding briefly:

> "Setup confirmado:
> - Tipo: [X]
> - AI modifier: [yes/no]
> - Stage: [stage]
> - Cadence: [cadence]
> - Critical context: [summary or 'pendente']
>
> Vou gerar os artefatos agora. Demora ~1-2 minutos."

Then proceed to Phase 2.

## Rules

- **Never ask follow-ups beyond these 5.** If the user asked you to elaborate, fine — answer their question, but don't proactively add Q6.
- **Never push back on "não sei".** Each skip behavior is defined; just apply it and move on.
- **Don't repeat the audit.** The user has already seen the discovery report. Reference it, don't re-show.
- **Order can be reordered** if the user starts answering Q3 before Q1 — adapt. The order above is a suggestion, not a script.
- **Language follows user's lead.** PT-BR if user is in Portuguese; EN if in English.

## What NOT to ask in Phase 1

The temptation will be to interrogate. Resist:

- ❌ "Por que você escolheu Drizzle ao invés de Prisma?"
- ❌ "Quais são os usuários do produto?"
- ❌ "Qual o roadmap dos próximos 6 meses?"
- ❌ "Quais decisões arquitetônicas mais importantes nessa codebase?"
- ❌ "Como funciona o fluxo de signup?"

Reasons not to ask:
- The first three are the user's job to fill in (or not) over time. Asking front-loads work.
- The last two are extractable from code (let Phase 0 do its job) or belong to PRD which is thin-by-design.

If the user wants to volunteer this info, capture it. But the skill never asks.
