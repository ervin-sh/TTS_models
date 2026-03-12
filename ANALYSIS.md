# TTS Experiment Analysis — Open Source vs AWS Nova Sonic v2

---

## 1. Overview — What Was Tested and Why

AWS Nova Sonic v2 represents the current high watermark for production-grade real-time voice AI: full-duplex, ultra-low latency, emotionally expressive, and watermarked for responsible deployment. The question driving this experiment was simple: **how close has the open-source ecosystem gotten, and what can you actually build with it today?**

Thirteen open-source models were evaluated against Nova Sonic v2 as the reference point. Testing was conducted on a local Mac wherever possible, with the same input text run through each model. Success was measured across four axes:

- **Naturalness** — does it sound like a person or a machine?
- **Latency** — can it keep up with real-time conversation?
- **Feature parity** — voice cloning, emotions, multi-speaker, watermarking
- **Practical viability** — can it run locally, is it commercially licensable, is setup reasonable?

The results were more interesting than expected.

---

## 2. Real-Time Voice Applications — The 300ms Boundary

Human conversation has a perceptual threshold around 300ms. Below it, a response feels instantaneous. Above it, the delay becomes noticeable — not crippling, but present. For voice AI agents, anything above 500ms starts to feel broken. This is the constraint that separates "interesting demo" from "deployable product."

### Latency Tiers Across Tested Models

| Tier | Threshold | Models |
| --- | --- | --- |
| **Tier 1 — Real-time capable** | < 200ms | Maya1 (sub-100ms, vLLM), CosyVoice 3 (~150ms), Chatterbox (<200ms), PersonaPlex (170ms turn-taking / 240ms interruption) |
| **Tier 2 — Borderline** | 200–300ms | VibeVoice-Realtime (~300ms) |
| **Tier 3 — Not real-time** | > 300ms or unspecified | CSM-1B (slow), fishspeech on Mac (very slow), DiVA (not specified), Dia-2B (not specified), Kokoro-82M (fast but unquantified), Magpie (fast but unquantified) |

### Full-Duplex vs Streaming — A Critical Distinction

Only two models in this set achieve **true full-duplex**: Nova Sonic v2 and PersonaPlex. Full-duplex means the model listens and speaks simultaneously — it can be interrupted mid-sentence, process that interruption, and respond. PersonaPlex achieves a 90.8% smooth turn-taking success rate and a 95.0% interruption success rate, which are the only published benchmarks for this capability in an open model.

Every other model operates in **half-duplex streaming mode** — they generate speech sequentially, and the system has to manually manage turn-taking at the application layer. This is workable for voice agents but adds engineering complexity (VAD, silence detection, barge-in logic).

**Practical implication:** For a voice AI phone agent or real-time voice assistant, CosyVoice 3 or Chatterbox are the strongest open-source candidates today. For anything requiring true conversational interruption handling, PersonaPlex is the only open option — but it requires a Linux machine with an NVIDIA GPU.

### What Real-Time Unlocks

- **AI call agents** — outbound sales, scheduling, support, with sub-200ms response
- **Live narration** — streaming audio for news, sports commentary, live translations
- **Voice-first interfaces** — hands-free device control, accessibility tools
- **Real-time dubbing** — Tier 1 latency + voice cloning (Chatterbox, CosyVoice 3) makes this feasible
- **Interactive fiction / games** — NPC voice lines generated dynamically, no pre-recording

---

## 3. The Efficiency Paradox — Smallest Dataset, Best Results

The most striking finding from this experiment was not which model had the most data or the most parameters. It was which model delivered the best result per unit of resource invested.

### The Kokoro-82M Finding

Kokoro-82M is the smallest model tested by a significant margin: **82 million parameters**, trained on **less than a few hundred hours** of audio. By conventional machine learning intuition, it should be the worst performer. It was not.

Personal assessment: **"Very natural — surprising."**

It runs blazing fast on a Mac, offers 54 pre-defined voices across 9 languages, is fully Apache 2.0 licensed, and was built for roughly $1,000 of compute. The architecture — StyleTTS 2 with an ISTFTNet vocoder — is doing more work than the raw parameter count suggests.

### Comparing Across the Scale Spectrum

| Model | Params | Training Data | Personal Result |
| --- | --- | --- | --- |
| Kokoro-82M | 82M | < few hundred hours | Very natural — surprising |
| Magpie TTS | 357M | 50,000 hours | Very natural — surprising |
| VibeVoice | 0.5B | Not disclosed | Surprisingly natural — almost the same feel as Nova Sonic 2 |
| Chatterbox | 0.5B | 500,000 hours | Very natural when tuned correctly |
| CSM-1B | 1B | Not disclosed | Sounds great |
| fishspeech S1 mini | 0.5B | 2,000,000+ hours | Sounds funny |
| DiVA | 8B | CommonVoice (size undisclosed) | Ugly, robotic |

The pattern is clear: **more data and more parameters do not reliably produce better-sounding speech.** fishspeech trained on over 2 million hours — roughly 40× Magpie's dataset — and produced a result that was rated worse than the 357M model with 50K hours. DiVA, at 8B parameters, was the most computationally expensive locally-runnable model and produced the worst audio.

The differentiator is **architecture and data curation**. Kokoro's training data was hand-selected from CC-licensed and public domain sources. Magpie used well-known, curated multilingual datasets (Hi-FiTTS, LibriTTS, CML-TTS). fishspeech used a large but internally unlabeled proprietary corpus. Scale without curation is expensive noise.

**Secondary confirmation:** Magpie achieves a CER of 0.34% on LibriTTS test-clean — a state-of-the-art result — with a model smaller than most on this list and a dataset that is fully disclosed and reproducible.

---

## 4. The Emotion and Affect Tagging Landscape

One of the most fragmented areas across the tested models was emotion control. Five fundamentally different paradigms were observed, each with different implications for how you'd integrate them into a production system.

### The Five Paradigms

| Paradigm | Models | Mechanism | Best For |
| --- | --- | --- | --- |
| **Inline tag injection** | Nova Sonic v2, CosyVoice 3, Maya1 | Insert `[laugh]`, `[sigh]`, `[excited]` / `<laugh>`, `<whisper>` directly in text | LLM-driven pipelines |
| **Parenthetical markers (49 tags)** | fishspeech S1 mini | `(laughing)` `(sobbing)` `(whispering)` `(angry)` `(sad)` `(excited)` `(proud)` `(disgusted)` + 40 more | Scripted, highly-controlled audio production |
| **Continuous exaggeration slider** | Chatterbox | CFG dial controls emotional intensity, Pace dial controls delivery speed | Character audio design, audiobooks, games |
| **Fine-grained vocal parameter control** | Ming-Omni | rate, pitch, volume, emotion, dialect as independent knobs | Studio-quality voice design workflows |
| **Emotion recognition (not generation)** | Kimi-Audio | SER — detects and classifies emotion in input audio | Audio analytics, sentiment analysis, not TTS |

### Analysis

**The inline injection model (Nova Sonic, CosyVoice 3, Maya1) is the most composable** for LLM pipelines. You can prompt an LLM to output `[laugh] I can't believe that worked` and pass it directly to TTS — the LLM becomes the emotion director. This is how Nova Sonic v2 was designed: the language model decides when to laugh, and the voice model executes it. CosyVoice 3 and Maya1 both replicate this pattern in open models. Maya1 adds a second dimension: natural language voice descriptions that let you specify a voice entirely in prose rather than selecting from a catalog.

**fishspeech's 49-tag vocabulary is the richest palette available**, offering tags that go well beyond simple emotion into affect and tone: `(whispering)`, `(panting)`, `(groaning)`, `(crowd laughing)`. In principle this enables nuanced voice performance. In practice, the base voice quality was rated as "sounds funny" in testing — which limits how much the emotional tags help. A well-delivered neutral voice is worth more than 49 tags on a voice that doesn't sound right.

**Chatterbox's exaggeration slider is distinct** because it controls intensity rather than type. You don't pick "angry" — you set how expressive the model should be and it interprets the text's emotional content. This is closer to how a voice director actually works ("give me 30% more intensity on that line"). Combined with its voice cloning capability, this makes Chatterbox interesting for character-level audio design in games or interactive applications.

**The open gap:** No model in this set does continuous, real-time emotion modulation during streaming. Every system requires the emotion to be decided before or during text generation — not during audio synthesis. Building a model that can shift emotional tone mid-stream in response to live conversational context is an open research problem.

---

## 5. Multi-Persona and Dialogue Systems

Generating convincing multi-speaker audio is one of the hardest TTS problems — it requires maintaining distinct voice identities across a conversation without them bleeding into each other. Four distinct approaches emerged from this experiment.

### Approach 1 — Architecture-First Dialogue (Dia-2B)

Dia-2B was designed from the ground up for two-speaker dialogue. The `[S1]` and `[S2]` speaker tags are not a bolted-on feature — they are part of the model's core training objective. Combined with the Kyutai Mimi codec and word-level timestamps, it can begin generating audio before the full script is complete (streaming-compatible generation).

Personal result: **"Multiple persona dialogue sounds quite natural."**

This makes it the strongest option for:
- Scripted podcast generation (give it a transcript, get a podcast)
- Synthetic dialogue dataset generation for training other models
- Audiobook production with multiple characters

The limitation is the 2-minute audio cap and English-only support. A single Dia-2B generation pass cannot produce a full podcast episode, but batched or chained generation can.

### Approach 2 — Context-Carry Conversation (CSM-1B)

CSM-1B takes a different approach: rather than speaker tags, it maintains conversational context across turns, and voice identity is preserved through context windows. The voice adapts to what was said before — a different philosophy from Dia-2B's structured tagging.

Personal result: **"Sounds great."**

This is closer to how a real voice AI agent should behave — the voice is not just consistent, it is contextually aware. CSM-1B also supports multiple speaker IDs for multi-party conversations and is fine-tunable, meaning you can specialize it for a specific persona. The downside is that it is slow on Mac and does not have published performance metrics.

### Approach 3 — Long-Form Broadcast (VibeVoice multi-speaker variant)

VibeVoice's separate multi-speaker TTS variant supports up to 90 minutes of continuous generation in a single pass. This is not a real-time conversational model — it is a batch production tool. Feed it a long transcript with multiple speakers marked, and it produces a complete audio file.

Personal result: **"Surprisingly natural — almost the same feel as Nova Sonic 2."**

For podcast production pipelines, documentary narration, or generating large volumes of training data, this is the most scalable option tested.

### Approach 4 — Zero-Shot Persona Cloning (Chatterbox, CosyVoice 3)

Both Chatterbox and CosyVoice 3 support zero-shot voice cloning from a short audio reference clip. Give either model 5–10 seconds of a target speaker, and it will generate new speech in that voice.

CosyVoice 3 extends this cross-lingually: clone a voice in English, then generate output in German, French, Japanese, or Korean — in that person's voice. This is the most powerful voice persona capability in the open-source set. Speaker similarity is benchmarked at 77.4–78%, and WER after RL tuning is 1.68% (English).

Combined with Chatterbox's emotion exaggeration control, you can in theory: clone a voice, set emotional intensity, and generate an expressive character performance — fully open-source, MIT licensed, runs on Mac.

---

## 6. Open Source vs Cloud — Where the Gap Actually Is

After testing, the gap between open-source TTS and Nova Sonic v2 is narrower than expected in some dimensions and wider than expected in others.

### Naturalness — Gap Narrowing

VibeVoice-Realtime-0.5B — a 0.5B model running on a local Mac — was rated as **"surprisingly natural — almost the same feel as Nova Sonic 2."** Kokoro-82M and Magpie were both rated "very natural — surprising." Maya1 was rated "very nice — ElevenLabs quality," running via MPS/CPU on Mac. These are not consolation prizes. For a significant fraction of use cases, the audio quality difference is not perceptible to the average listener.

### Latency — Partial Parity

CosyVoice 3 at ~150ms and Chatterbox at <200ms are within the real-time conversation window. They are not as fast as Nova Sonic v2's "ultra-low" streaming, but they are fast enough for practical voice agent applications.

### Full-Duplex — Still a Cloud Moat

True simultaneous listen-and-speak is only available via Nova Sonic v2 (cloud) and PersonaPlex (Linux/NVIDIA). No Mac-compatible open model achieves this. Building interruption handling with open models requires additional application-layer engineering.

### Responsible AI — The Watermarking Gap

This is the most significant gap. Nova Sonic v2 claims built-in watermarking. Among open models, only **VibeVoice** (imperceptible watermark + audible AI disclaimer) and **Chatterbox** (Perth watermarking, survives MP3 compression and audio editing with ~100% detection) have addressed this. Every other open model generates audio with no attribution mechanism.

For any commercial deployment of AI-generated voice — customer service, media, education — this gap matters from both regulatory and ethical standpoints.

### The License Trap

One hidden finding: **fishspeech S1 mini is CC-BY-NC-SA-4.0 (non-commercial only)**. It appears on HuggingFace alongside fully open models but cannot be used commercially. For a builder who discovers this after integration, it is a significant problem. Every other model in this experiment (excluding Nova Sonic v2) is Apache 2.0, MIT, or NVIDIA's open commercial license.

---

## 7. What This Enables — Build Ideas Grounded in What Actually Works

These are not hypothetical. Each build is based directly on demonstrated capabilities and personal test results.

### Local Voice AI Agent Stack
**Stack:** Kokoro-82M or Magpie + any open STT model
**Why:** Both rated "very natural — surprising," both run on Mac, both commercially free. Combined with a streaming STT, this is a sub-300ms voice agent pipeline with zero cloud dependency.
**Gap to fill:** Barge-in / interruption handling must be built at the application layer.

### Expressive Character Audiobook Pipeline
**Stack:** Chatterbox (voice cloning + emotion exaggeration)
**Why:** Clone each character's voice from a short reference clip, use the exaggeration slider to differentiate performance intensity by character, generate entire chapters in sequence.
**Result ceiling:** "Very natural when you tune the correct exaggeration and weights" — requires calibration per character.

### Synthetic Dialogue Dataset Factory
**Stack:** Dia-2B
**Why:** `[S1]` / `[S2]` tags, natural-sounding dialogue output ("sounds quite natural"), word-level timestamps for alignment. Generate thousands of synthetic two-speaker conversations for fine-tuning conversational AI, TTS, or ASR models.
**Scale note:** 2-minute cap per generation, English only.

### Cross-Lingual Voice Preservation
**Stack:** CosyVoice 3
**Why:** Zero-shot cross-lingual voice cloning at 77.4–78% speaker similarity. Clone a speaker's voice from English recordings, generate content in German, French, Japanese, Korean, or any of the 9 supported languages.
**Use case:** Dubbing a podcast, localizing an e-learning course in the instructor's own voice, voice preservation for speakers who have lost their voice.

### AI Podcast Production at Scale
**Stack:** VibeVoice multi-speaker TTS variant
**Why:** 90-minute generation in a single pass, multiple speakers, rated nearly equivalent to Nova Sonic 2 in naturalness.
**Use case:** Automated podcast creation from text scripts, synthetic media production, large-scale audio content pipelines.

### Responsible AI Deployment
**Stack:** VibeVoice (dual watermarking) or Chatterbox (Perth watermarking)
**Why:** The only open models with provenance tracking for AI-generated audio. VibeVoice adds both an imperceptible digital watermark and an audible "this audio was generated by AI" disclaimer. Chatterbox's Perth watermark survives MP3 encoding, audio editing, and mixing — critical for detecting misuse.
**Use case:** Any production deployment where regulatory compliance or brand accountability for AI-generated voice matters.

---

## Key Takeaways

1. **Architecture beats scale.** Kokoro-82M (82M params, ~$1,000 training) outperformed fishspeech (0.5B params, 2M+ hours of data) on naturalness. Dataset curation and model design are the real differentiators.

2. **The 300ms wall is crossable.** Three open models are under it on their own hardware. Real-time voice AI does not require a cloud API.

3. **Full-duplex is still the cloud moat.** Interruption handling is the one thing open-source has not solved at scale.

4. **Emotion is a fragmented, unsolved design space.** Five different paradigms exist with no consensus. Inline tag injection (Nova Sonic, CosyVoice 3, Maya1) is the most composable for LLM pipelines; nobody has solved real-time mid-stream emotion modulation yet.

5. **Multi-persona is closer than expected.** Dia-2B and CSM-1B both produce convincing multi-speaker audio. Chatterbox and CosyVoice 3 make arbitrary voice personas possible from short clips.

6. **Watermarking is a critical gap.** 11 of 13 open models ship with no audio provenance mechanism. This will matter more, not less, as AI voice generation becomes mainstream.

7. **Check the license before you build.** fishspeech looks open and has impressive capabilities on paper, but its CC-BY-NC-SA-4.0 license makes it unusable for commercial products.
