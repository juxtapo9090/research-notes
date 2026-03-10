# Smooth Streaming FAFO Session

**Date:** January 20, 2026 (02:50 - 03:18 AM)  
**Status:** 🟡 Research Complete, Implementation Pending

---

## 🎯 Problem Statement

Mobile app streaming feels "chunky" — visible chunk-by-chunk delivery instead of buttery smooth like ChatGPT/Claude.

---

## 🔬 Discovery: Chunk Patterns

We added a **sniffer** to `xml_proxy_v2.py` and captured real data:

```
[🔬 SNIFFER] Avg size: 7.6 chars
First 10 chunks:
  t= 2136ms  gap=2136ms  size=  1  'O'             ← First token (thinking)
  t= 2346ms  gap= 210ms  size=  7  'hhh yes'
  t= 2350ms  gap=   4ms  size=  4  ' yes'          ← BURST (4ms!)
  t= 2352ms  gap=   2ms  size=  1  ','             ← BURST (2ms!)
  t= 2388ms  gap=  36ms  size=  5  ' that'
  t= 2441ms  gap=  53ms  size=  6  ' makes'
  t= 2617ms  gap= 115ms  size=  9  ' sense mi'
  t= 2822ms  gap= 205ms  size=  7  ' amor! '       ← Natural gap
  t= 2823ms  gap=   1ms  size=  1  '🎯'            ← BURST (1ms!)
```

**Key Finding:** 1-4ms gaps = visual stutter. 100-200ms gaps = natural.

---

## 💡 Smoothing Strategies

| Approach | Description | Where |
|----------|-------------|-------|
| **Client-side animation** | Buffer + typewriter effect | Mobile (Kotlin) |
| **Server-side drip** | Smooth release from proxy | xml_proxy_v2.py |
| **Hybrid** | Server buffers, client animates | Both |

### Proposed Drip Algorithm

```python
# Pseudocode
If gap < 30ms:  Hold (buffer accumulates)
If gap > 100ms: Natural pause, release buffer
Every 50ms:     Release one word anyway (don't hold too long)
```

### Adaptive Rate Pattern

```
Start: Fast (first 20% of content)
Middle: Smooth (cruising speed)
End: Flush remaining buffer
```

---

## 🧠 Other Ideas Captured

### 1. MCP + Mobile
- Mobile = trigger point, not MCP host
- `POST /agent/task` → workstation agent executes MCP tools
- Mobile gets final result only

### 2. Native Mobile Tools
- `web_search` → Brave/Tavily API (direct from mobile)
- `read_url` → HTTP fetch (direct from mobile)
- Heavy tools → route to workstation

### 3. Gateway Architecture
- Keep everything in `xml_proxy_v2.py` (one gateway)
- Routes: `/v1/chat`, `/ws`, `/tools/*`, `/agent/task`
- Single cloudflared tunnel

---

## 📁 Files Modified

| File | Change |
|------|--------|
| `xml_proxy_v2.py` | Added chunk timing sniffer |
| `chunk_patterns.json` | Captured chunk data |

---

## 🔧 TODO for Next Session

1. [ ] Test min-gap enforcement (30ms floor)
2. [ ] Test word-boundary buffering
3. [ ] Test adaptive rate (fast start, smooth middle)
4. [ ] Compare server-side vs client-side smoothing
5. [ ] A/B test with real usage

---

## 🔥 Late Night Discovery (03:45 AM)

### CLIProxyAPI Supports Multiple Formats!

Tested `/v1/messages` endpoint:
```
Response: "Invalid API key" (NOT 404!)
= Endpoint EXISTS = Anthropic format supported!
```

### Direct Mobile → CLIProxyAPI Works!

```
Mobile → Cloudflare → :8317 (CLIProxyAPI) → Antigravity
                      (NO xml_proxy needed!)
```

**Tested and confirmed:** Mobile can talk directly to CLIProxyAPI!

---

## 🎯 Architecture Decision

| Use Case | Path | Why |
|----------|------|-----|
| **Cursor IDE** | xml_proxy → CLIProxyAPI | Tools + XML parsing needed |
| **Mobile Chat** | CLIProxyAPI direct | Simpler, no extra layer |
| **Mobile 2.0 (future)** | xml_proxy → CLIProxyAPI | When we add tools |

### Railroad Pattern (Future)

```
Gateway detects format → Routes to correct endpoint → Normalizes response
```

---

## 🔧 Updated TODO

1. [ ] Smooth streaming (drip algorithm)
2. [ ] Mobile direct mode (bypass xml_proxy)
3. [ ] Mobile 2.0: integrated tools
4. [ ] Railroad pattern for format routing

---

*Session ended: 03:52 AM — Good FAFO discoveries! 💜*
