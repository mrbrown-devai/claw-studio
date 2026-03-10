# Inter-Agent Workflows

> Comment les agents collaborent sur les tâches majeures.

---

## WF-A: Nouveau Projet Client

**Déclencheur** : Lead qualifié détecté par Radar ou inbound

```
┌─────────┐     ┌─────────┐     ┌─────────┐     ┌─────────┐
│  Radar  │────▶│ Vector  │────▶│  Prime  │────▶│   Arc   │
│   📍    │     │   🎯    │     │   👁️   │     │   🤖    │
│ Detect  │     │ Qualify │     │ Approve │     │  Plan   │
└─────────┘     └─────────┘     └─────────┘     └─────────┘
                     │                              │
                     ▼                              ▼
              ┌─────────┐                    ┌─────────┐
              │ Keeper  │                    │ Anchor/ │
              │   🗂️   │                    │ Engine/ │
              │  Track  │                    │ Pixel   │
              └─────────┘                    └─────────┘
                                                  │
                                                  ▼
                                            ┌─────────┐
                                            │  Probe  │
                                            │   🔍    │
                                            │   QA    │
                                            └─────────┘
```

### Étapes

| Step | Agent | Action | Output | Gate |
|------|-------|--------|--------|------|
| 1 | Radar 📍 | Détecte un lead potentiel | Lead brief | - |
| 2 | Vector 🎯 | Qualifie le lead | Qualified/Pass | - |
| 3 | Vector 🎯 | Discovery call + proposal | Proposal doc | - |
| 4 | Vector 🎯 | Négociation | Deal terms | HUMAN |
| 5 | Prime 👁️ | Valide le deal | Approved | HUMAN |
| 6 | Keeper 🗂️ | Enregistre le client | CRM entry | - |
| 7 | Arc 🤖 | Crée l'architecture (RFC) | RFC doc | - |
| 8 | Team | Build le projet | Code | - |
| 9 | Probe 🔍 | QA + Testing | Test report | - |
| 10 | Sentinel 🛡️ | Security review (si mainnet) | Audit | HUMAN |
| 11 | Arc 🤖 | Delivery | Deployed | - |

---

## WF-B: Lancement Produit

**Déclencheur** : QA + Security APPROVED

```
┌─────────┐     ┌─────────┐     ┌─────────┐     ┌─────────┐
│  Probe  │────▶│Sentinel │────▶│   Arc   │────▶│  Echo   │
│   🔍    │     │   🛡️   │     │   🤖    │     │   📢    │
│QA Pass  │     │ Secure  │     │ Deploy  │     │Announce │
└─────────┘     └─────────┘     └─────────┘     └─────────┘
                                                     │
                     ┌───────────────────────────────┤
                     ▼               ▼               ▼
              ┌─────────┐     ┌─────────┐     ┌─────────┐
              │  Chirp  │     │  Molt   │     │  Pulse  │
              │   🐦    │     │   🐸    │     │   💬    │
              │   X     │     │  Memes  │     │   TG    │
              └─────────┘     └─────────┘     └─────────┘
```

### Étapes

| Step | Agent | Action | Output | Gate |
|------|-------|--------|--------|------|
| 1 | Probe 🔍 | Confirme QA passed | QA report | - |
| 2 | Sentinel 🛡️ | Security audit (mainnet) | Audit report | HUMAN |
| 3 | Arc 🤖 | Deploy to production | Deployment | HUMAN (mainnet) |
| 4 | Echo 📢 | Prépare announcement | Campaign brief | HUMAN |
| 5 | Chirp 🐦 | Post sur X | Thread | - |
| 6 | Molt 🐸 | Crée les memes | Memes | - |
| 7 | Pulse 💬 | Annonce TG | Message | - |
| 8 | Bridge 🌉 | Engage community | Replies | - |

---

## WF-C: Incident Sécurité

**Déclencheur** : Exploit détecté ou vulnérabilité critique

```
┌─────────┐     ┌─────────┐     ┌─────────┐
│Sentinel │────▶│  Prime  │────▶│  HUMAN  │
│   🛡️   │     │   👁️   │     │  GATE   │
│ Detect  │     │ Assess  │     │ Decide  │
└─────────┘     └─────────┘     └─────────┘
     │                              │
     ▼                              ▼
┌─────────┐                   ┌─────────┐
│   Arc   │                   │  Echo   │
│   🤖    │                   │   📢    │
│   Fix   │                   │  Comms  │
└─────────┘                   └─────────┘
```

### Étapes

| Step | Agent | Action | Output | Gate |
|------|-------|--------|--------|------|
| 1 | Sentinel 🛡️ | Détecte incident | Alert | - |
| 2 | Sentinel 🛡️ | Évalue severity | Severity report | - |
| 3 | Prime 👁️ | Coordonne response | Action plan | HUMAN |
| 4 | Arc 🤖 | Pause/fix contracts | Mitigation | - |
| 5 | Echo 📢 | Prépare communication | Statement | HUMAN |
| 6 | Chirp 🐦 | Public statement | Post | HUMAN |
| 7 | Bridge 🌉 | Community support | Replies | - |
| 8 | Sentinel 🛡️ | Post-mortem | Report | - |

**⚠️ CRITICAL : Tout incident = HUMAN GATE immédiat**

---

## WF-D: Campagne Marketing

**Déclencheur** : Brief de Echo ou opportunité détectée

```
┌─────────┐     ┌─────────┐     ┌─────────┐
│  Echo   │────▶│  Quill  │────▶│  Chirp  │
│   📢    │     │   ✍️    │     │   🐦    │
│ Brief   │     │ Content │     │  Post   │
└─────────┘     └─────────┘     └─────────┘
     │               │               │
     ▼               ▼               ▼
┌─────────┐     ┌─────────┐     ┌─────────┐
│  Prism  │     │  Molt   │     │  Pulse  │
│   💎    │     │   🐸    │     │   💬    │
│ Design  │     │ Memes   │     │   TG    │
└─────────┘     └─────────┘     └─────────┘
```

### Étapes

| Step | Agent | Action | Output | Gate |
|------|-------|--------|--------|------|
| 1 | Echo 📢 | Crée campaign brief | Brief doc | - |
| 2 | Quill ✍️ | Écrit le contenu | Copy/threads | - |
| 3 | Prism 💎 | Crée les visuels | Graphics | - |
| 4 | Molt 🐸 | Crée les memes | Meme pack | - |
| 5 | Echo 📢 | Review + approve | Approved | HUMAN (si major) |
| 6 | Chirp 🐦 | Post sur X | Posts | - |
| 7 | Pulse 💬 | Post sur TG | Messages | - |
| 8 | Bridge 🌉 | Engage + track | Metrics | - |

---

## WF-E: Intelligence → Décision

**Déclencheur** : Signal détecte une opportunité ou menace

```
┌─────────┐     ┌─────────┐     ┌─────────┐
│ Signal  │────▶│  Prime  │────▶│  Team   │
│   📡    │     │   👁️   │     │ Varied  │
│Research │     │ Decide  │     │  Act    │
└─────────┘     └─────────┘     └─────────┘
     ▲
     │
┌────┴────┐
│ Oracle  │     ┌─────────┐
│   🔮    │◀────│ Shadow  │
│On-chain │     │   🕵️   │
└─────────┘     │  Intel  │
                └─────────┘
```

### Étapes

| Step | Agent | Action | Output | Gate |
|------|-------|--------|--------|------|
| 1 | Oracle 🔮 | Monitor on-chain | Data | - |
| 2 | Shadow 🕵️ | Competitive intel | Intel report | - |
| 3 | Signal 📡 | Synthétise findings | Signal report | - |
| 4 | Prime 👁️ | Évalue et décide | Decision | HUMAN (if major) |
| 5 | Relevant agent | Execute decision | Action | - |

---

## Message Format Inter-Agent

```json
{
  "id": "msg_[uuid]",
  "from": "agent_id",
  "to": "agent_id",
  "workflow": "WF-X",
  "step": 1,
  "priority": "URGENT | NORMAL | LOW",
  "human_gate": false,
  "timestamp": "2026-03-10T12:00:00Z",
  "context": "Background information",
  "task": "What needs to be done",
  "inputs": {
    "key": "value"
  },
  "expected_output": {
    "format": "markdown | json | code",
    "deadline": "2026-03-11T12:00:00Z"
  }
}
```

---

## Escalation Path

```
Agent (AUTO) → Manager (SEMI) → C-Suite → Prime → HUMAN
```

Timeout escalation :
- 4h sans réponse → escalade au manager
- 24h sans réponse → escalade à Prime
- 48h sans réponse → HUMAN GATE automatique

---

*Claw Studio — Workflows v1.0*
