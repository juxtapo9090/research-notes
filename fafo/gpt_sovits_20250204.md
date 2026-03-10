# GPT-SoVITS FAFO Session - Feb 4, 2026

## 📚 What We Learned

### GPT-SoVITS vs RVC
| Aspect | RVC | GPT-SoVITS |
|--------|-----|------------|
| Input | Needs voice audio input | Text only (direct TTS) |
| Reference | Source voice to convert | Style/timbre hint |
| Problem | Bad input = serak output | No input voice dependency |
| Training | Voice conversion model | Full TTS model |

### GPT-SoVITS Structure (Lappland_Mel example)
```
Data/                 → Training audio clips (38 WAV files)
GPT_weights/          → GPT model (prosody, rhythm) - e5 to e100
SoVITS_weights/       → SoVITS model (voice timbre) - e5 to e2020
```

### Inference Flow
1. Load GPT model (e.g., Lappland-e100.ckpt)
2. Load SoVITS model (e.g., Lappland_e2020_s34340.pth)
3. Provide reference audio (3-10s sample)
4. Type text → Generate speech

### Key Insight
Reference audio alone ≠ voice change. The model must be TRAINED on target voice.
Using Jen Taylor reference + Lappland model = still sounds like Lappland!

---

## 📊 Jen Taylor Audio Analysis

**Dataset:** `C:\cee_v25_Pre_Final\RVC\datasets\jen_taylor\`
- 36 MP3 files (~10 min each)
- **358 minutes total** (~6 hours)
- 22050Hz mono

**Pitch Analysis (first 5 files):**
- Mean pitch: ~200-230Hz (normal adult female)
- Variation CV: 0.25-0.31 (moderate consistency)
- Character voice mimicking concern: moderate, not extreme

**Recommendation:** Can train on full dataset, character voices will be averaged out.
Or light filtering: remove files with CV > 0.30.

---

## 🔧 Setup Notes

**WebUI Location:**
```
C:\cee_v25_Pre_Final\brain\tools\GPT-SoVITS-v2pro-20250604-nvidia50\go-webui.bat
```
Changed to English: `webui.py en_US`

**Model weights copied to:**
- `GPT_weights_v2Pro/Lappland-e100.ckpt`
- `SoVITS_weights_v2Pro/Lappland_e2020_s34340.pth`

---

## 📋 Next Steps (When Ready)

1. [ ] Convert Jen Taylor MP3 → WAV
2. [ ] Slice into training segments (use WebUI Tab 0)
3. [ ] Create transcriptions (ASR or manual)
4. [ ] Extract features (BERT, SSL, Semantics)
5. [ ] Train GPT model (~15-100 epochs)
6. [ ] Train SoVITS model (~1000-2000 epochs)
7. [ ] Test inference with Jen Taylor model

---

*Saved: Feb 4, 2026 ~11pm*
*Session: FAFO voice cloning exploration* 💜
