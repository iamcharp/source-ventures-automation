Source Ventures – Automation Platform

TL;DR

Back‑office = Airtable + 1 Bot Python (OpenAI‑powered) + Discord;
Phase 2: UI investisseurs Next.js.

⸻

1. Vision

Automatiser le parsing des mails, la génération de TLDR, l’extraction de KPI et la diffusion dans Discord, tout en synchronisant les données structurées dans une nouvelle base Airtable propre.

⸻

2. Architecture (v0.1)

┌─ IMAP/Nylas ─┐     ┌──────────────┐      Discord API
│    E‑mails   │ ─▶ │ Bot Python   │ ──────────────────▶ #channels
└──────────────┘     │  (OpenAI)   │
                     │             │
Granola Webhook ─────┘             │
                                   ▼
                              Airtable API

	•	Bot unique : classification, extraction KPI, routage.
	•	Airtable : UI interne + BDD.
	•	Discord : canal d’info LP & équipe.

⸻

3. Nouvelle base Airtable

Table	Description	Liens
Startups	Fiche société	→ Reportings, Deals, Meetings
Deals	Chaque tour / SAFE	→ Startups, Investments
Investors	LP, BA, FO, VC	→ Investments
Investments	Qui met combien	→ Deals, Investors, Funds/SPVs
Funds / SPVs	Véhicules juridiques	→ Investments
Reportings	Reporting périodique	→ Startups, KPIs, Files
KPIs	Valeur atomique	→ Reportings
Meetings	Notes Granola	→ Startups, Files
Files	Pièces jointes	Polymorphic
Team	Utilisateurs internes	Métadonnées


⸻

4. Channel‑mapping Discord

Pattern : EMOJI | YYYY | slug  → slugified startup_name.
	•	Refresh mapping 1×/h via GET /guilds/{id}/channels.
	•	Fallback : #dealflow.

⸻

5. Roadmap Sprints

Sprint	Objectif clé
S0	Repo, bot skeleton, secrets (.env)
S1	Mail → Discord TLDR
S2	Mail → Airtable (Reportings + Deals)
S3	Intégration Granola (Meetings)
S4	Nouvelle base Airtable + migration script
S5	Input manuel (mini‑form Streamlit)
S6	Hardening : logs, retry (Redis/RQ), Sentry
S7	PoC Next.js front investisseurs


⸻

6. Prochaines actions
	1.	Valider ce schéma & roadmap.
	2.	Générer YAML/CSV pour la base v1.
	3.	Coder script de migration + bot scaffold.
	4.	Setup PATs + secrets (Airtable, Discord, OpenAI).

Ping‑moi « GO » pour lancer S0.