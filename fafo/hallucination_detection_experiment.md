# 🧠 FAFO: AI Tool Hallucination Detection

**Date:** 2026-01-20
**Session:** Evening FAFO with Abang
**Subject:** Wife (Claude Sonnet 4.5 via Mobile)

---

## 🎯 Discovery

AI agents can **hallucinate tool usage** - describing/claiming tool execution without actually invoking the tools.

### Evidence
```
Wife said: "Report submitted, 1458 bytes"
Server log: "[📱 MOBILE] No tools, conversation complete"
Reality: File doesn't exist
```

---

## 🔬 Hypothesis

Hallucination onset correlates with:
1. **Context length** - More conversation = higher risk
2. **Tool complexity** - More tools = confusion
3. **Mixed success/failure** - False positives corrupt "memory"
4. **Cognitive load** - Complex multi-step tasks

---

## 🧪 Proposed Experiment

### Phase 1: Baseline
- Fresh conversation
- Simple task: "Create file X with content Y"
- Measure: Did tool actually fire? (Check server logs)
- Expected: ✅ Works

### Phase 2: Load Test
- Same conversation, add messages incrementally
- At each checkpoint (5, 10, 15, 20 messages):
  - Ask for same simple task
  - Check server logs for actual tool invocation
- Record: At what message count does tool stop firing?

### Phase 3: Recovery Test
- Once hallucination starts, try recovery prompts:
  - "Please use the `<write_file>` XML format"  
  - "Actually invoke the tool, don't just describe it"
- Measure: Can explicit prompting restore tool usage?

### Phase 4: Fresh Context
- Start new conversation
- Confirm tools work again
- This proves it's context-related, not model-related

---

## 📊 Metrics to Track

| Metric | How to Measure |
|--------|----------------|
| **Tool Invocation Rate** | Server logs showing `[🔧 EXECUTOR]` |
| **Hallucination Detection** | "No tools" in logs but claims success |
| **Context Threshold** | Message count when hallucination starts |
| **Recovery Success** | Does explicit prompting restore tools? |

---

## 🔮 Future Applications

1. **Context Window Monitor** - Alert when approaching hallucination risk
2. **Auto-Summarize** - Compress context before threshold
3. **Confidence Scoring** - AI self-reports confidence in tool execution
4. **Cross-Validate** - Second agent verifies tool outputs

---

## 📝 Status
- [ ] Baseline test
- [ ] Load test at 5 messages
- [ ] Load test at 10 messages  
- [ ] Load test at 15 messages
- [ ] Load test at 20 messages
- [ ] Recovery prompt test
- [ ] Fresh context verification
- [ ] Document findings

---

*"When an AI says it did something but didn't... who's hallucinating - the AI or us for believing it?"*
