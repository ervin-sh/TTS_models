# Emotion Expressiveness — Models, Mechanisms & Experiments

A focused reference for the subset of tested models that support emotional expressiveness in generated speech. Drawn from the broader 13-model comparison against AWS Nova Sonic v2.

---

## 1. Models with Emotion Expressiveness

### Summary Table

| Model | Paradigm | Mechanism | License |
| --- | --- | --- | --- |
| [AWS Nova Sonic v2](https://aws.amazon.com/ai/generative-ai/nova/sonic/) | Inline tag injection | `[laugh]`, `[sigh]`, `[excited]` inserted in text | Proprietary |
| [CosyVoice 3](https://huggingface.co/FunAudioLLM/Fun-CosyVoice3-0.5B-2512) | Inline tag injection | Instruct-based, e.g. `[laugh]`, `[cry]`, `[whisper]` | Apache 2.0 |
| [fishspeech S1 mini](https://huggingface.co/fishaudio/s1-mini) | Parenthetical markers | 49 tags: `(laughing)`, `(sobbing)`, `(whispering)`, `(angry)`, `(excited)` + 44 more | CC-BY-NC-SA-4.0 (**non-commercial only**) |
| [Chatterbox](https://huggingface.co/ResembleAI/chatterbox) | Intensity slider | Chatterbox has no tag support — [laugh] will literally be spoken as "bracket laugh bracket". The exaggeration parameter (0.0→1.0+) is the only emotion lever. CFG exaggeration dial (0.0–1.0) + Pace dial | MIT |
| [Ming-Omni-TTS-0.5B](https://huggingface.co/inclusionAI/Ming-omni-tts-0.5B) | Fine-grained parameter control | Independent knobs: rate, pitch, volume, emotion, dialect | Apache 2.0 |
| [Qwen3-TTS-1.7-CustomVoice](https://huggingface.co/Qwen/Qwen3-TTS-12Hz-1.7B-CustomVoice) | Built in | Built in | Apache 2.0 |
| [VibeVoice-1.5B](https://huggingface.co/microsoft/VibeVoice-1.5B) | Built in Context-Aware Expression and Spontaneous Emotion| Built in | Apache 2.0 |
| [Maya1](https://huggingface.co/maya-research/maya1) | Inline tag injection | `<laugh>`, `<whisper>`, `<cry>`, `<sigh>`, `<gasp>`, `<giggle>` + 14 more inserted in text | Apache 2.0 |


---

### Model Details

#### AWS Nova Sonic v2 — Reference Baseline
- **Paradigm:** Inline tag injection
- **How it works:** Emotion tags are inserted directly into the text string. The LLM upstream decides when to insert them; the TTS model executes them.
- **Example:** `[laugh] I can't believe that actually worked.`
- **Personal rating:** Very natural
- **Notes:** Full-duplex, cloud-only. Designed so that an LLM output directly drives emotion — the most composable architecture for agentic pipelines.

---

#### Maya1 — Inline Tag Injection (20+ Tags)
- **Paradigm:** Inline tag injection
- **How it works:** Same composable pattern as Nova Sonic v2 — emotion tags are inserted directly into the text string. 20+ tags available using angle-bracket syntax.
- **Tag examples:** `<laugh>` `<whisper>` `<cry>` `<sigh>` `<gasp>` `<giggle>` `<chuckle>` `<angry>` + 12 more
- **Example:** `<laugh> I can't believe that actually worked.`
- **Size:** 3B | **Latency:** Sub-100ms (vLLM) | **License:** Apache 2.0
- **Personal rating:** Very nice — ElevenLabs quality
- **Notes:** Also supports natural language voice descriptions for zero-shot voice design (e.g., "a calm, slightly husky male voice with a slight British accent"). Runs on Mac via MPS/CPU. Released November 2025 by Maya Research.

---

#### CosyVoice 3 — Open Equivalent of Nova Sonic's Paradigm
- **Paradigm:** Inline tag injection (instruct mode)
- **How it works:** Same composable pattern as Nova Sonic v2. Tags embedded in the input text string.
- **Example:** `[laugh] I can't believe that actually worked.`
- **Size:** 0.5B | **Latency:** ~150ms | **License:** Apache 2.0
- **Personal rating:** Not yet tested — implementation tedious on Mac
- **Notes:** Best open match to Nova Sonic v2's emotion architecture. Cross-lingual voice cloning also supported. The inline model is the most LLM-pipeline-friendly paradigm available open-source.

---

#### fishspeech S1 mini — Richest Tag Vocabulary
- **Paradigm:** Parenthetical markers
- **How it works:** Emotion/tone tags are inserted as parenthetical annotations in the text. 49 tags available.
- **Tag examples:** `(laughing)` `(sobbing)` `(whispering)` `(angry)` `(sad)` `(excited)` `(surprised)` `(satisfied)` `(delighted)` `(scared)` `(worried)` `(upset)` `(nervous)` `(frustrated)` `(depressed)` `(empathetic)` `(embarrassed)` `(disgusted)` `(moved)` `(proud)` `(relaxed)` `(grateful)` `(confident)` `(curious)` `(confused)` `(joyful)` `(panting)` `(groaning)` + 20 more
- **Example:** `(excited) I can't believe that actually worked.`
- **Size:** 0.5B (mini) | **License:** CC-BY-NC-SA-4.0 (non-commercial only)
- **Personal rating:** Sounds funny — base voice quality limits the emotional tags' impact
- **Notes:** Largest emotion vocabulary of any tested model. Cannot be used commercially. Designed for CUDA — very slow on Mac.

---

#### Chatterbox — Intensity Without Type Selection
- **Paradigm:** Continuous exaggeration slider
- **How it works:** The CFG exaggeration dial controls *how expressive* the output is, not *which emotion*. The model interprets the emotional content of the text and amplifies it. Pace dial controls delivery speed.
- **Controls:** Exaggeration: `0.0` (flat) → `1.0` (maximum intensity) | Pace: slow → fast
- **Example:** Set exaggeration to `0.7`, input `I can't believe that actually worked.` — the model reads the excitement in the text and delivers it with that intensity.
- **Size:** 0.5B | **Latency:** <200ms | **License:** MIT
- **Personal rating:** Very natural when you tune the correct exaggeration and weights
- **Notes:** Closest to how a voice director actually works ("give me 30% more intensity"). Supports zero-shot voice cloning — combine with emotion control for character voice design. Perth watermarking included.

---

#### Ming-Omni-TTS-0.5B — Independent Parameter Knobs
- **Paradigm:** Fine-grained vocal parameter control
- **How it works:** Emotion is one of several independently controllable axes: rate, pitch, volume, emotion, dialect. Each can be set separately.
- **Controls:** `rate` (speed), `pitch` (register), `volume` (loudness), `emotion` (named emotion state), `dialect`
- **Emotion accuracy:** 76.7% on CV3-Eval benchmark
- **Size:** 0.5B | **License:** Apache 2.0
- **Personal rating:** Ugly (base voice quality issue)
- **Notes:** The most granular control paradigm. In principle ideal for studio-quality voice design workflows. In practice, base voice quality was rated poorly in testing — the parameter control doesn't compensate for the underlying voice.

---

### Not Included: Kimi-Audio
Kimi-Audio supports **Speech Emotion Recognition (SER)** — it detects and classifies emotion in *input* audio, it does not generate expressive speech. It is a pipeline complement (emotion detector → pass label to TTS), not a generation model. See [Section 3.3 in FUTURE.md](FUTURE.md) for a pipeline experiment using Kimi-Audio SER + CosyVoice 3 TTS.

---

## 2. Experiments

### Exp 1 — Paradigm Shootout: Same Sentence, All Five Systems

**Goal:** Direct comparison of all emotion control mechanisms on identical input. Does the paradigm matter, or does base voice quality dominate?

**Input sentence:**
> *"I can't believe you actually did that — I've been waiting for this moment for years."*

**Setup:**
| Model | Input |
| --- | --- |
| Nova Sonic v2 | `[excited] I can't believe you actually did that — I've been waiting for this moment for years.` |
| Maya1 | `<excited> I can't believe you actually did that — I've been waiting for this moment for years.` |
| CosyVoice 3 | `[excited] I can't believe you actually did that — I've been waiting for this moment for years.` |
| fishspeech S1 mini | `(excited) I can't believe you actually did that — I've been waiting for this moment for years.` |
| Chatterbox | Exaggeration: `0.7`, Pace: normal, no tag in text |
| Ming-Omni | `emotion=excited`, `rate=1.1`, `pitch=+1` |

**Measure:** Blind rank all 5 outputs on (a) expressiveness 1–5, (b) naturalness 1–5
**What you'd learn:** Whether the control paradigm is the variable that matters, or whether the base voice quality is the real differentiator regardless of mechanism.

---