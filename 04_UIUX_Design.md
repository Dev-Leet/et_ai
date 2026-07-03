# UI/UX Design Document
## Unified Asset & Operations Brain

---

## 1. Design Principles

1. **Field-first, not desk-first** — the primary user is often a technician on a plant floor with gloves, poor lighting, and one-handed device use.
2. **Trust through transparency** — every AI answer visibly shows its source and confidence; no "black box" answers.
3. **Low cognitive load** — industrial users need answers fast, not dashboards to interpret.
4. **Progressive disclosure** — simple answer first, "show sources / show reasoning" expandable.
5. **Accessibility** — WCAG 2.1 AA, large touch targets, high-contrast mode for outdoor/bright environments.

---

## 2. User Personas

| Persona | Context | Primary Need |
|---|---|---|
| Rahul, Field Technician | On plant floor, mobile, noisy environment | Quick answer to "how do I do X on equipment Y", voice input |
| Priya, Maintenance Engineer | Desk + occasional field | RCA workflows, failure history, predictive alerts |
| Amit, Compliance Officer | Desk, audit prep season | Compliance gap dashboard, evidence package export |
| Sunita, Plant Manager | Desk, review meetings | Cross-functional insight dashboard, KPI trends |

---

## 3. Information Architecture

```
Home / Dashboard
├── Knowledge Copilot (Chat)
│   ├── Query input (text/voice)
│   ├── Answer + citations + confidence
│   └── Conversation history
├── Maintenance Intelligence
│   ├── Equipment 360° view
│   ├── RCA workspace (guided workflow)
│   └── Predictive maintenance calendar
├── Compliance Center
│   ├── Compliance gap dashboard
│   ├── Audit evidence package generator
│   └── Regulatory mapping browser
├── Lessons Learned
│   ├── Incident/near-miss feed
│   ├── Pattern insights
│   └── Proactive alerts
├── Knowledge Graph Explorer (power users)
├── Document Library (ingestion status, source browser)
└── Admin
    ├── User & role management
    ├── Ingestion connectors
    └── Audit log
```

---

## 4. Key Screens

### 4.1 Knowledge Copilot (Primary Screen — Mobile & Desktop)
- Chat-style interface, large input field, mic icon for voice
- Each answer card shows: answer text → confidence badge (High/Medium/Low, color-coded) → source document chips (tap to open) → "Was this helpful?" feedback
- Low-confidence answers show a banner: "This may need expert review" with an escalate button

### 4.2 Equipment 360° View
- Tabs: Overview | Maintenance History | Documents | Failure Patterns | Related Procedures
- Timeline visualization of work orders and inspections
- "Ask Copilot about this equipment" contextual shortcut

### 4.3 RCA Workspace
- Guided stepper: Problem statement → Contributing factors (auto-suggested from history) → Root cause → Corrective action
- Side panel shows relevant historical incidents pulled automatically

### 4.4 Compliance Gap Dashboard
- Card grid grouped by regulation (Factory Act / OISD / PESO / Environmental)
- Each gap card: severity (color), affected equipment/procedure, recommended action, "Generate Evidence Package" button

### 4.5 Document Library
- Table view: filename, type, ingestion status, extracted entity count, last updated
- Upload drag-and-drop zone; batch upload support

---

## 5. Mobile-Specific UX Requirements

- Copilot chat must work one-handed, thumb-reachable input
- Offline mode: cached recent procedures/answers available without connectivity; queued queries sync when back online
- Voice-first interaction for hands-busy scenarios (PPE, gloves)
- Minimal data usage mode (text-only, no heavy images) for low-bandwidth plant zones

---

## 6. Visual Design System

| Element | Guidance |
|---|---|
| Color Palette | Neutral industrial base (slate/graphite) + high-contrast accent for alerts (amber/red) and confidence (green/amber/red) |
| Typography | Clear sans-serif (e.g., Inter), minimum 16px body on mobile |
| Iconography | Consistent line-icon set for equipment, documents, compliance, alerts |
| Confidence Indicators | Green (High ≥85%), Amber (Medium 60–84%), Red (Low <60%, requires review) |
| Dark/Light Mode | Both supported; high-contrast "field mode" for outdoor visibility |

---

## 7. Interaction Patterns

- **Citation-first answers:** citations always visible, never hidden behind a click for the first source
- **Confidence transparency:** no answer is shown without a confidence indicator
- **Escalation loop:** low-confidence or safety-critical answers route to a human-review queue with one tap
- **Contextual shortcuts:** "Ask Copilot" available from any equipment/document/incident screen

---

## 8. Usability & Accessibility Targets

- WCAG 2.1 AA compliance (contrast ratios, keyboard navigation, screen-reader labels)
- Touch targets ≥ 44x44px on mobile
- Support for Hindi + English UI (localization-ready strings)
- Voice input/output for low-literacy or hands-busy users
