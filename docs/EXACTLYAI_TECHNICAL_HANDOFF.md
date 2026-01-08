# ExactlyAI Dev Handoff: POC → Production Architecture

**Created:** 2026-01-08
**Purpose:** Bridge POC work (Claude Code CLI) to Dev Team production systems
**Source:** Exactly Architecture Writeup (Google Doc)

---

## 1. Exactly Production Architecture (From Dev Team)

### Frontend (User-Facing)
| Service | Stack | Hosting |
|---------|-------|---------|
| Exactly Website | NextJS/React | exactlyai.solutions |
| Exactly Portal | Internal management | - |
| Chatbot UI | Reusable component | - |
| Chatbot Demo Sites | Demo deployments | Vercel + Google Cloud Run |

### Backend (APIs & Orchestrators)
| Service | Stack | Purpose |
|---------|-------|---------|
| TS API | Bun, Hono, Vercel AI SDK | Primary API, agent orchestration |
| API Manager | Go/Gin | Queueing, caching, client interactions |
| Exactly Backend | Go, Elixir, Gleam | Core orchestration |
| Crawl Service | Python + TypeScript | Web crawling |
| Prospective Services | POCs | Rapid prototyping |

### Data Layer
| Database | Purpose |
|----------|---------|
| PostgreSQL (exactly-dev) | Development |
| PostgreSQL (exactly-prod) | Production |

### Planned Components
- **Exactly MCP** - Centralized tool repository
- **Data Lake** - Unified data source
- **Service Bus** - Inter-service communication
- **Event Hub** - Notification propagation

---

## 2. POC Work Completed (Claude Code CLI)

### Demo Chatbot System
| Component | Location | Status |
|-----------|----------|--------|
| Signature Systems Demo | `demo_site_generator/demo_sites/signature-systems/` | Production |
| Niagara Conservation Demo | `demo_site_generator/demo_sites/niagara-conservation/` | Production |
| Chatbot JS (v4.2.0) | `shared/chatbot.js` | Deployed |
| API Proxy | `exactlyai-api-proxy.vercel.app` | Production |

### Chatbot Configuration
| File | Purpose |
|------|---------|
| `demo_config.json` | Smart Open triggers, NEPQ phases, qualification rules |
| `chatbot.js` | Runtime behavior, embedded knowledge, API calls |
| `EXACTLYAI_CHATBOT_VOICE_AUTHORITY.md` | Voice/behavior rules, editor agent spec |

### Intelligence Database (SQLite)
**File:** `exactlyai_industry_mapping.db` (272 KB)

| Table | Records | Purpose |
|-------|---------|---------|
| industry_mapping | 647 | LinkedIn → Apollo translation, B2B scores |
| competitor_taxonomy | 14 | Pattern matching, risk levels |
| ideal_client_profiles | 10 | A-D tier definitions |
| exclusion_rules | 10 | HARD/SOFT exclude rules |
| company_evaluations | 612+ | Scraped company data |
| scrape_log | - | Learning entries |
| industry_url_effectiveness | - | Pattern tracking |
| scrape_patterns | 16 | URL patterns by industry |

### Lead Pipeline (CLI Bridge)
| Component | File | Purpose |
|-----------|------|---------|
| Blackjack Scraper | `cli_bridge/blackjack_scraper.py` | Multi-page website scraping |
| Scoring System | `cli_bridge/score_companies.py` | A/B/C/X grade assignment |
| Apollo Integration | `apollo_lead_extractor.py` | People search, email reveal |

---

## 3. Integration Points: POC → Production

### Chatbot Demo Sites → Production
```
POC (Current)                    Production (Target)
─────────────────────────────────────────────────────
chatbot.js (inline knowledge)  → Chatbot UI component
demo_config.json               → Portal configuration
Vercel proxy                   → TS API / API Manager
Claude Haiku                   → Vercel AI SDK orchestration
```

### Knowledge Pipeline → Crawl Service
```
POC (Current)                    Production (Target)
─────────────────────────────────────────────────────
blackjack_scraper.py           → Crawl Service (Python/TS)
EXACTLYAI_CONFIG.context       → Data Lake (client knowledge)
SQLite tables                  → PostgreSQL (exactly-prod)
```

### Two-Layer Prompt Architecture (Proposed)
```
┌─────────────────────────────────────────────────────────┐
│  BASELINE LAYER (Managed by Dev Team)                  │
│  ─────────────────────────────────────────────────────  │
│  • Brand voice / personality                           │
│  • Behavioral rules                                    │
│  • Conversation methodology                            │
│  • Quality guardrails                                  │
│                                                        │
│  Location: TS API / Exactly Backend                    │
└─────────────────────────────────────────────────────────┘
                          +
┌─────────────────────────────────────────────────────────┐
│  CLIENT LAYER (Generated per-client)                   │
│  ─────────────────────────────────────────────────────  │
│  • Products & services (scraped)                       │
│  • Company facts                                       │
│  • Industry context                                    │
│                                                        │
│  Location: Data Lake / PostgreSQL                      │
└─────────────────────────────────────────────────────────┘
```

---

## 4. Data Structures for Dev Team

### Client Knowledge Schema (JSON)
```json
{
  "company": {
    "name": "string",
    "website": "string",
    "phone": "string",
    "industry": "string",
    "audience_type": "b2b | b2c | mixed"
  },
  "products": [
    {
      "name": "string",
      "description": "string",
      "specs": {
        "dimensions": "string",
        "weight": "string",
        "capacity": "string",
        "material": "string"
      },
      "ideal_for": ["string"],
      "price_range": "string (optional)"
    }
  ],
  "services": ["string"],
  "facts": {
    "certifications": ["string"],
    "notable_clients": ["string"],
    "locations": ["string"],
    "founded": "string (optional)"
  },
  "selection_guide": [
    {
      "need": "string",
      "recommendation": "string"
    }
  ],
  "contact": {
    "phone": "string",
    "email": "string (optional)",
    "quote_url": "string (optional)"
  }
}
```

### Chatbot Configuration Schema (JSON)
```json
{
  "smart_open": {
    "triggers": {
      "delay_seconds": "number",
      "scroll_depth_pct": "number",
      "exit_intent": "boolean",
      "idle_seconds": "number"
    }
  },
  "voice": {
    "mode": "mitch | professional | custom",
    "forbidden_phrases": ["string"],
    "max_sentence_length": "number"
  },
  "qualification": {
    "target_industries": ["string"],
    "disqualify_signals": ["string"],
    "routing_rules": [
      {
        "audience": "string",
        "action": "continue | redirect | different_greeting",
        "destination": "string"
      }
    ]
  },
  "llm": {
    "model": "claude-3-haiku-20240307 | claude-sonnet-4-20250514",
    "max_tokens": "number",
    "response_length": "short | medium | long"
  }
}
```

### Company Evaluation Table (PostgreSQL)
```sql
CREATE TABLE company_evaluations (
    id SERIAL PRIMARY KEY,
    company_name VARCHAR(255) NOT NULL,
    website VARCHAR(512),
    industry VARCHAR(100),
    employee_count INTEGER,

    -- Scoring
    grade CHAR(1),  -- A, B, C, X
    score INTEGER,

    -- Scraped Data
    homepage_content TEXT,
    about_content TEXT,
    services_content TEXT,
    leadership_content TEXT,

    -- Validation
    is_b2b BOOLEAN,
    is_competitor BOOLEAN,
    has_chatbot BOOLEAN,

    -- Metadata
    website_scraped_date TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

---

## 5. Pending Decisions (Dev Team Input Needed)

| Item | Question | POC Default |
|------|----------|-------------|
| LLM Model | Haiku vs Sonnet for production? | Haiku |
| Voice/NEPQ | Keep current methodology or replace? | NEPQ + Mitch Mode |
| Editor Agent | Implement Generate→Critique→Patch? | Spec complete, not built |
| Knowledge Storage | Inline JS vs API fetch vs DB? | Inline JS |
| Baseline Prompt | Dev team to provide from testing? | Current NEPQ prompt |

---

## 6. Files to Transfer

### Core Documentation
- `EXACTLYAI_CHATBOT_VOICE_AUTHORITY.md` - Voice rules, editor spec, architecture
- `EXACTLYAI_CHATBOT_FILE_INDEX.md` - File inventory
- `demo_site_generator/demo_sites/signature-systems/demo_config.json` - Config example

### Working Code
- `demo_site_generator/demo_sites/signature-systems/shared/chatbot.js` - Reference implementation
- `exactlyai-api-proxy/` - Vercel proxy for Claude API
- `cli_bridge/blackjack_scraper.py` - Website scraping pipeline

### Database
- `exactlyai_industry_mapping.db` - SQLite (migrate to PostgreSQL)

---

## 7. Recommended Next Steps

1. **Dev Team:** Review two-layer architecture proposal
2. **Dev Team:** Provide updated baseline prompt from testing
3. **Dev Team:** Decide on LLM model (Haiku vs Sonnet)
4. **POC Team:** Migrate SQLite schemas to PostgreSQL format
5. **Integration:** Connect Crawl Service output to client knowledge schema

---

## Document History

| Date | Change |
|------|--------|
| 2026-01-08 | Initial creation from Google Doc architecture review |
