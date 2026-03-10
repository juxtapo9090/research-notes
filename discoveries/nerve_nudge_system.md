# Nerve Nudge System - Staged Alert Injection

**Discovered:** 2026-01-20
**Context:** FAFO brainstorming - AI hallucination prevention via background nudging

---

## Core Concept

DefCon-style severity levels for AI correction, injected behind the scenes.

```
Julia Observer → Detects Hallucination → Writes Warning
       ↓
xml_proxy_v2.py → Reads Warning → Generates Nudge
       ↓
AI Context → Receives Grounding → Corrects Course
```

## Severity Levels

| Level | Emoji | Inject As | Template |
|-------|-------|-----------|----------|
| **CAUTION** | 🟡 | system | Gentle reminder to verify |
| **WARNING** | 🟠 | system | Firm correction, use tools |
| **ALERT** | 🔴 | user | HALT - force verification |

## Key Design Decisions

1. **ALERT injects as user message** - AI cannot ignore user messages
2. **CAUTION/WARNING inject as system** - Subtle background guidance
3. **One-shot consumption** - Warning is deleted after processing
4. **History logging** - All nudges logged to `nudge_history.jsonl`

## Integration Point

```python
# In mobile_chat():
warning = nerve_nudge.check_for_warning()
if warning:
    nudge = nerve_nudge.generate_nudge(warning)
    messages = nerve_nudge.inject_nudge(messages, nudge)
```

## Julia's Warning Format

```json
{
  "severity": "WARNING",
  "reason": "Multiple turns without tool verification",
  "context": "Turns 3, 4, 5 all had hallucination risk",
  "source": "Julia Nerve Observer"
}
```

## Files

- `xml_proxy_v2.py` - NerveNudgeSystem class
- `brain/nerve_inbox/active_warning.json` - Julia writes here
- `brain/nerve_inbox/nudge_history.jsonl` - Consumption log
- `brain/test_nudge_system.py` - Test script

## FAFO Results

- ✅ 3-tier severity working
- ✅ Nudges inject correctly based on severity
- ✅ History tracking operational
- 🔄 Next: Connect Julia observer to write warnings automatically

---

## Future Enhancement: Session Metrics HUD

**Requires:** Mobile APK modification

### Concept

Display model health/reliability metrics in response header:

```
claude-sonnet-4.5 | 🟢 87% stability | T=0.7 | Ctx: 45%
Hello! I'm just finishing X and Y...
```

### Proposed Metrics

| Metric | Source | Purpose |
|--------|--------|---------|
| **Stability %** | `nudge_history.jsonl` | Clean turns vs hallucinations ratio |
| **Context %** | Token count / max | Memory utilization |
| **Temp** | Request body | Current creativity level |
| **Tool Rate** | `chat_logs` | Tool execution success rate |
| **Session Health** | Consecutive clean | Drift detection |
| **Quota %** | `quota_bridge` | Account capacity |

### Implementation Ideas

```python
class SessionMetrics:
    def calculate_stability(self, n=20):
        """Stability = clean_turns / total_turns * 100"""
        logs = self.load_recent(n)
        clean = sum(1 for l in logs if l['risk'] in ['LOW', 'UNKNOWN'])
        return int((clean / len(logs)) * 100) if logs else 100
    
    def get_health_emoji(self, stability):
        if stability >= 80: return "🟢"
        if stability >= 50: return "🟡"
        return "🔴"
```

### Integration Options

1. **Server-side prepend** - `xml_proxy` adds header to response
2. **APK header bar** - Kotlin UI shows metrics
3. **Both** - Server calculates, APK displays

### When to Build

- After nudge system proven in production
- When APK gets UI refresh
- Collect baseline data first

---

**Tags:** #hallucination #nudge #injection #nerve-system #defcon #fafo #future
