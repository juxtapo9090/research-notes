# Distributed Brain Architecture
> FAFO Design Session - 2026-01-23 Night

## 🧠 Core Insight

The Brain can be **distributed across VPS + PC**, with each component running where it makes sense:
- **VPS** = Nerve endings (lightweight catchers)
- **PC** = Brain matter (heavy Flux processing)
- **xml_proxy** = The arbiter that merges insights into conversation

---

## Key Concepts

### 1. Lite vs Full Brain
| Version | Where | Dependencies | Purpose |
|---------|-------|--------------|---------|
| `brain.jl` | PC | Flux, ML heavy | Pattern learning, prediction |
| `brain_lite.jl` | VPS | Just JSON3 | Rule-based hallucination detection |

**Why**: Flux needs ~2GB RAM minimum. VPS has 1GB. Keep ML on PC.

### 2. Catch & Release Pattern
```
VPS catches event → writes to SSHFS → PC processes with Flux → releases result → xml_proxy injects
```
The neural highway is **file-based via SSHFS** - zero new networking needed.

### 3. Brain Provider Ceremony
Like task_boundary but for brain components:
```julia
brain_boundary(
    Provider = "flux_watcher",
    Mode = "ANALYSING",       # CATCHING | ANALYSING | RELEASING | DORMANT
    Confidence = 0.85,
    AwaitingFrom = ["xml_proxy"]
)
```
Each provider announces its state, capabilities, and what it can contribute.

### 4. Latency-Biased Routing
Providers race against a **time budget**:
- **< 50ms**: Instant injection (local rules, nerve_lite)
- **< 100ms**: Include if ready (qdrant memory)
- **> 100ms**: Queue for NEXT turn (flux pattern analysis)

Fast providers WIN the right to influence THIS turn. Slow providers contribute to FUTURE turns.

```
User sends message
├── [0-50ms]   fast providers → INJECT NOW
├── [50-100ms] medium providers → INJECT IF READY  
├── [100ms]    ═══ CUTOFF: Send to Claude ═══
│
Claude thinking (~2-10s)
├── [meanwhile] slow providers analyze previous turn
│
Claude responds
└── [next turn] inject queued slow insights
```

### 5. xml_proxy as Final Arbiter
All brain insights converge at xml_proxy:
- **Pre-flight**: Gather fast insights before sending to Claude
- **Post-flight**: Queue slow analyses for next turn
- **Session state**: V3 server-side history keeps context

### 6. Squid vs xml_proxy
| Squid | xml_proxy |
|-------|-----------|
| HTTP traffic | LLM traffic |
| Doesn't parse body | Parses tools, injects context |
| Stateless caching | Stateful conversation |
| Network layer | Application layer |

**Conclusion**: xml_proxy is the right tool for AI conversation orchestration.

---

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    DISTRIBUTED BRAIN                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   VPS (1GB RAM)                PC Workstation (16GB+)       │
│   ─────────────                ──────────────────────       │
│   xml_proxy_v2.py              flux_watcher.jl              │
│   nerve_lite.jl                brain.jl + Flux              │
│   Iris daemon                  pattern_net.jl               │
│                                                              │
│        │              SSHFS               │                 │
│        └──────────────────────────────────┘                 │
│                     (file sync)                             │
│                                                              │
│   catch/ ─────────────────────────────► process/            │
│   release/ ◄───────────────────────── results/              │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Next Steps (Not Implemented Yet)
1. Create `flux_watcher.jl` on PC that polls catch directory
2. Add brain provider registry (`heartbeat.json`)
3. Refactor xml_proxy with middleware stack
4. Add latency budget to pre-flight injection

---

## Quotes from Session

> "Flux will kill the VPS -_-"

> "The brain that can respond in time WINS the right to influence the next turn"

> "Normal internet traffic vs LLM traffic - dua dunia berbeza"

---

*Filed to: discoveries/ (architectural insight)*
*Session: FAFO Night 2026-01-23*
