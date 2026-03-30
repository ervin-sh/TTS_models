# Future Experiments — TTS Research Roadmap

Building on the initial model comparison, these are the next experiments worth running. Grouped by goal, not by model. Each entry states what to test, what to measure, and what you'd learn from it.

---

## 1. Multi-Persona & Speaker Identity

The initial experiments established that multi-speaker audio is possible with open models. The open questions are about robustness, consistency, and how far you can push identity control.

---

### 1.1 Blind Listening Shootout — Same Transcript, Three Approaches
**Goal:** Find which multi-speaker system produces the most convincingly distinct voices.

**Setup:**
- Write a single 2-minute, 2-speaker dialogue transcript
- Generate it with Dia-2B (`[S1]`/`[S2]`), CSM-1B (speaker IDs), and VibeVoice multi-speaker variant
- Play all three to listeners without labels — ask them to rate how distinct the two speakers sound (1–5) and which felt most natural

**Measure:** Perceived speaker distinctiveness score, naturalness score per output

**What you'd learn:** Whether the architectural approach (Dia-2B's baked-in dialogue design) outperforms context-carry (CSM-1B) in a controlled head-to-head.

---

### 1.2 Character Voice Consistency Under Non-Sequential Generation
**Goal:** Test whether a cloned voice stays stable across many separate generation calls.

**Setup:**
- Clone one voice with Chatterbox from a 10-second reference clip
- Generate 30 lines of dialogue spread across 3 separate sessions (not one continuous generation)
- Run speaker similarity scoring (WavLM or ECAPA-TDNN) between the reference and each output

**Measure:** Speaker similarity score across all 30 outputs; variance between sessions

**What you'd learn:** Whether voice cloning is reliable enough for a production audiobook pipeline where chapters are generated independently over days.

---

### 1.3 Long-Form Persona Drift
**Goal:** Measure how speaker identity degrades over extended generation.

**Setup:**
- Generate a continuous 60-minute audio file with VibeVoice multi-speaker TTS variant
- Segment it into 5-minute chunks
- Score speaker similarity of each segment against the first 5 minutes (baseline)

**Measure:** Speaker similarity vs time curve

**What you'd learn:** At what point (if any) does the model's voice drift away from the original identity. This directly informs how to chunk long-form production safely.

---

### 1.4 Cross-Model Persona Chain
**Goal:** Test whether you can pass a cloned voice identity across model boundaries.

**Setup:**
- Clone a voice with Chatterbox → export a generated audio clip
- Feed that clip as the reference voice into CosyVoice 3 → generate new text in a different language
- Compare the final output to the original reference speaker

**Measure:** Speaker similarity between original reference and final cross-language output; subjective naturalness

**What you'd learn:** Whether cross-lingual voice cloning chains work in practice — relevant for multilingual AI dubbing workflows.

---

### 1.5 Speaker Tag Swap — Does the Model Follow Identity or Content?
**Goal:** Test how much the model's internal speaker representations bias the output.

**Setup:**
- Write a script where S1 is clearly assertive/dominant and S2 is hesitant
- Generate normally, then swap all `[S1]` and `[S2]` tags and regenerate
- Listen: does the emotional character follow the tag, or does the model push back based on line content?

**Measure:** Subjective assessment of whether the swapped output sounds natural or wrong

**What you'd learn:** How tightly voice identity and linguistic content are entangled in Dia-2B's architecture.

---

## 2. Emotion Tagging & Affect Control

Six different paradigms exist for emotion control across the tested models. None of them have been directly compared on the same input. That's the experiment.

---

### 2.1 Paradigm Shootout — Same Sentence, All Five Systems
**Goal:** Direct comparison of every emotion control mechanism on identical input.

**Setup:**
- Choose one emotionally rich sentence (e.g., *"I can't believe you actually did that — I've been waiting for this moment for years"*)
- Generate with:
  - Nova Sonic v2: `[excited] I can't believe you actually did that...`
  - CosyVoice 3: `[excited] I can't believe you actually did that...`
  - fishspeech: `(excited) I can't believe you actually did that...`
  - Chatterbox: exaggeration slider at 0.7, pace normal
  - Ming-Omni: emotion=excited, rate=1.1, pitch=+2
  - Voxtral-4B-TTS: voice preset `cheerful_female`, no tag in text
- Blind rank all 6 on expressiveness and naturalness

**Measure:** Ranked preference scores; which system "sells" the emotion most convincingly

**What you'd learn:** Whether the paradigm (tag vs slider vs parameter) matters, or whether base voice quality dominates the perception.

---

### 2.2 Chatterbox Intensity Calibration — Finding the Uncanny Valley
**Goal:** Map the exaggeration slider to perceptual quality.

**Setup:**
- Generate the same emotionally neutral sentence with Chatterbox at exaggeration values: 0.1, 0.2, 0.3, 0.5, 0.7, 0.9, 1.0
- Rate each on a scale: sounds natural / slightly exaggerated / over the top / uncanny
- Repeat for a clearly emotional sentence

**Measure:** The threshold value at which the output stops sounding like a person

**What you'd learn:** The usable range of the exaggeration control in practice. The Chatterbox card says to tune it — this tells you exactly where the ceiling is.

---

### 2.3 LLM-Directed Emotion Injection — No Human Tagging
**Goal:** Test whether an LLM can serve as a competent emotion director.

**Setup:**
- Take a short screenplay (8–10 lines of dialogue) with no emotion tags
- Prompt Claude to read it and output the same text with emotion tags inserted (using CosyVoice 3 or fishspeech syntax)
- Generate the audio
- Compare against: (a) no tags, (b) human-tagged version of the same script

**Measure:** Subjective preference between LLM-tagged, human-tagged, and untagged outputs

**What you'd learn:** Whether LLMs can be trusted to handle emotion direction in a fully automated pipeline, or whether human review is still required.

---

### 2.4 Tag vs. Content Conflict — Who Wins?
**Goal:** Test whether emotion tags override the semantic content of the text.

**Setup:**
- Sentence A (happy content): *"We just got the funding — we're celebrating tonight!"* → tag as `(sad)`
- Sentence B (sad content): *"He passed away quietly in his sleep last night."* → tag as `(excited)`
- Generate both with fishspeech and CosyVoice 3
- Listen: does the tag win, or does the content win?

**Measure:** Subjective assessment of which signal dominates

**What you'd learn:** The reliability of emotion tags as a control mechanism — critical for scripted production pipelines where the tag must take precedence over the literal words.

---

### 2.5 Emotion Stability in Long-Form Speech
**Goal:** Test whether a set emotion tag holds across a 2-minute monologue.

**Setup:**
- Write a 300-word neutral monologue (factual, descriptive content)
- Generate with fishspeech at `(sad)` throughout
- Segment into 30-second clips — rate each for emotional consistency with the tag

**Measure:** Whether the affect drifts toward neutral as the generation continues

**What you'd learn:** Whether emotion tags are sticky across long outputs or whether they decay — important for audiobook production and long-form narration.

---

## 3. STT + LLM + TTS — Full Pipeline Experiments

The individual TTS models have been tested in isolation. The next step is testing complete voice agent loops: speech in, speech out, with LLM reasoning in the middle.

---

### 3.1 Round-Trip Latency Benchmark
**Goal:** Measure the total latency of a locally-runnable voice agent pipeline, broken down by stage.

**Setup:**
```
Mic → [STT] → [LLM] → [TTS] → Speaker
```
- STT: faster-whisper (base.en)
- LLM: Claude via API or a local model (e.g., Llama 3.1 8B via Ollama)
- TTS: Kokoro-82M (fastest local option)
- Record timestamps at each stage boundary

**Measure:**
- STT latency (audio end → text ready)
- LLM latency (first token → response complete)
- TTS latency (text in → first audio chunk out)
- Total end-to-end from speech end to audio start

**What you'd learn:** Which stage is the actual bottleneck. Based on current data, TTS is likely not the constraint — LLM and STT probably dominate.

---

### 3.2 STT Bottleneck Study
**Goal:** Identify the fastest STT model that still produces clean enough text for a TTS pipeline.

**Setup:**
- Record the same 60-second audio clip (clean speech)
- Transcribe with: Whisper tiny.en, base.en, small.en, faster-whisper base, Moonshine (if available locally)
- Measure: transcription time, WER (compare to ground truth transcript)

**Measure:** Latency vs WER tradeoff curve

**What you'd learn:** The cheapest STT configuration that doesn't introduce errors that would make the downstream LLM or TTS output wrong. Determines the realistic floor for pipeline latency.

---

### 3.3 Emotion-Aware Pipeline
**Goal:** Build a pipeline that detects the speaker's emotional state and responds with appropriately matched TTS affect.

**Setup:**
```
Voice input → Kimi-Audio SER (classify emotion) →
LLM (system prompt: "user sounds [detected emotion], respond empathetically") →
CosyVoice 3 (output tagged with [matching emotion])
```
- Test with recordings of the same sentence delivered in angry, happy, sad, and neutral tones
- Rate: does the system's emotional response feel appropriate to the input?

**Measure:** Subjective appropriateness score for each emotional input

**What you'd learn:** Whether real-time emotion detection (Kimi-Audio SER) → LLM → emotionally matched TTS is a viable pipeline, or whether the emotion detection is too coarse to be useful.

---

### 3.4 Persona-Locked Multi-Turn Conversation
**Goal:** Test whether a voice agent maintains a consistent character voice across a full multi-turn conversation.

**Setup:**
- CSM-1B as TTS (context-aware, multi-turn)
- STT: faster-whisper
- LLM: Claude with a system prompt defining a character persona
- Run 10 turns of conversation
- After each turn, score speaker similarity of TTS output vs turn 1

**Measure:** Speaker similarity curve across 10 turns; subjective consistency rating

**What you'd learn:** Whether CSM-1B's context-carry mechanism actually preserves voice identity in a dynamic conversation, or whether the voice drifts as the context window fills.

---

### 3.5 Voice-to-Voice Translation
**Goal:** Hear yourself speaking a language you don't know, in your own voice.

**Setup:**
```
Record English sentence → faster-whisper STT →
Claude API: translate to [German / French / Japanese] →
CosyVoice 3: generate translation with zero-shot clone of input voice
```
- Use the STT-transcribed audio as the reference clip for voice cloning
- Listen to the output: does it sound like you speaking another language?

**Measure:** Subjective naturalness, speaker similarity to original recording

**What you'd learn:** End-to-end viability of a real-time voice translation pipeline using only open models + Claude API. CosyVoice 3's cross-lingual cloning (71.8–78% speaker similarity) is the key capability being stress-tested here.

---

### 3.6 Barge-In Simulation
**Goal:** Test how quickly the pipeline can abort mid-speech and restart.

**Setup:**
- TTS model begins playing audio
- At T+1s, inject a new STT input while audio is still playing
- Measure: time from new input detection to audio stop → new response start

**Measure:** Abort-and-restart latency; subjective "did it feel responsive?" rating

**What you'd learn:** The engineering feasibility of barge-in (user interrupts the AI mid-sentence) using open models. This is the last major gap vs Nova Sonic v2's full-duplex capability.

---

## 4. Watermarking & Responsible AI

Two open models have built-in audio watermarking. Neither has been tested under adversarial conditions.

---

### 4.1 Perth Watermark Stress Test (Chatterbox)
**Goal:** Establish how much post-processing the Chatterbox watermark can survive.

**Setup:**
- Generate 10 audio clips with Chatterbox
- Apply each of the following transformations independently:
  - MP3 compression: 320kbps, 192kbps, 128kbps, 64kbps
  - Background noise addition (white noise, cafeteria noise) at -10dB, -20dB
  - Pitch shift: ±1, ±2, ±3 semitones
  - Speed change: 0.9×, 1.1×, 1.25×
  - Audio editing: cut and re-join 3 segments
- Run Perth detection on each result

**Measure:** Detection rate (%) per transformation type and severity

**What you'd learn:** The real operational robustness of the watermark. Chatterbox claims ~100% detection — this tests whether that holds under realistic post-production conditions.

---

### 4.2 VibeVoice Audible Disclaimer Detectability
**Goal:** Find out whether non-expert listeners notice the VibeVoice audible AI disclaimer without being told to listen for it.

**Setup:**
- Generate 5 audio clips with VibeVoice
- Play them to 10 listeners with no context — just "listen to this audio clip"
- Ask: "Did you notice anything unusual?"
- Then ask: "Was this AI generated?"

**Measure:** % of listeners who detected the disclaimer unprompted; % who correctly identified AI generation

**What you'd learn:** Whether the audible watermark is actually providing transparency to end listeners, or whether it's so subtle that it only serves automated detection systems.

---

## 5. Systematic Quality Benchmarking

The initial testing was subjective — same person, same setup, impressionistic ratings. These experiments replace that with reproducible numbers.

---

### 5.1 UTMOS Automated Quality Scoring
**Goal:** Get a reproducible MOS estimate for every model on the same test set.

**Setup:**
- Define a 20-sentence benchmark set covering: simple statements, complex sentences, questions, emotionally charged lines, technical vocabulary
- Generate all 20 with every model
- Score all outputs with UTMOS (automated MOS predictor)

**Measure:** UTMOS score per model; correlation with subjective ratings already recorded

**What you'd learn:** Whether the subjective ratings ("very natural", "sounds funny", "ugly robotic") hold up as reproducible scores. Also enables future comparisons as new models emerge.

---

### 5.2 WER Intelligibility Test
**Goal:** Measure how accurately each model reproduces the input text in audio form.

**Setup:**
- Generate 50 sentences of increasing complexity with each locally-runnable model
- Run all outputs through Whisper (base.en) → compare to input text → calculate WER
- Sentence set: 10 simple, 10 moderate, 10 with numbers/dates, 10 with proper nouns, 10 with complex grammar

**Measure:** WER per model per sentence category

**What you'd learn:** Which models garble specific types of content (numbers, names, complex structure). Critical for use cases like reading financial data, medical text, or technical documentation.

---

### 5.3 Mac Hardware Speed Benchmark
**Goal:** Get reproducible tokens-per-second or characters-per-second numbers for all locally-runnable models.

**Setup:**
- Same machine (M-series Mac, standardized)
- Generate the same 500-character input 5 times per model
- Record wall-clock time from input to audio file complete
- Calculate effective characters/second

**Measure:** Generation speed in chars/sec; real-time factor (audio duration / generation time)

**What you'd learn:** Actual speed differences between models — Kokoro-82M "blazing fast" and Magpie "extremely fast" are both unquantified. This turns them into numbers you can plan a pipeline around.

---

## 6. Synthetic Data Generation

These experiments treat the TTS models as data factories rather than end products — using their output to build something larger.

---

### 6.1 Dialogue Dataset Factory
**Goal:** Generate a large synthetic 2-speaker conversation dataset for fine-tuning.

**Setup:**
- Use Claude to generate 1,000 short dialogue scripts (8–12 lines each, varied topics, tones, and situations)
- Feed each through Dia-2B → 1,000 audio files with S1/S2 word-level timestamps
- Package as a dataset: audio + transcript + speaker timestamps

**Measure:** Total generation time, audio quality spot-check on 50 random samples, WER on 100 samples

**What you'd learn:** Whether Dia-2B is a viable synthetic data engine for training conversational ASR or TTS models. The resulting dataset could be used to fine-tune a smaller, faster dialogue model.

---

### 6.2 Multi-Speaker Voice Corpus
**Goal:** Build a synthetic voice corpus with diverse speaker identities.

**Setup:**
- Source 10 reference voice clips (public domain, varied age/gender/accent)
- Clone each with Chatterbox
- Generate 500 sentences per voice → 5,000 utterances total
- Each utterance labeled with: speaker ID, text, emotion tag (neutral), generation timestamp

**Measure:** Speaker similarity across all 500 utterances per voice; WER on 10% sample

**What you'd learn:** Whether Chatterbox's zero-shot cloning is consistent enough to serve as a synthetic TTS data source — and whether the resulting corpus is high-quality enough to train or fine-tune another model.

---

### 6.3 Emotion-Labeled Speech Corpus
**Goal:** Generate a labeled dataset for training a speech emotion recognition model.

**Setup:**
- fishspeech S1 mini × 49 emotion/tone tags × 20 sentences = 980 labeled utterances
- Each utterance: audio file + text + emotion label
- Cross-check: run Kimi-Audio SER on the outputs — does it classify them correctly?

**Measure:** SER classification accuracy on fishspeech-generated emotion-labeled data; agreement between fishspeech tag and Kimi-Audio SER prediction

**What you'd learn:** Whether synthetic emotion-labeled TTS data (from fishspeech) is clean enough to train a downstream SER model — closing the loop between TTS emotion generation and SER emotion recognition.

---

## 7. Novel & Creative Experiments

---

### 7.1 Ambient + Speech — Ming-Omni's Unique Capability
**Goal:** Test whether Ming-Omni's unified speech + ambient sound generation sounds immersive or artificial.

**Setup:**
- Prompt Ming-Omni to generate a 60-second podcast-style monologue with embedded ambient sounds: light keyboard typing in background, occasional paper shuffle, room ambience
- Compare against: same text generated with Kokoro-82M, then ambient sounds mixed in post-production with Audacity

**Measure:** Subjective: which sounds more naturally integrated? Which sounds more like a real recording?

**What you'd learn:** Whether in-model ambient generation (Ming-Omni) is a genuine advantage over post-processing, or whether traditional audio mixing produces a more natural result.

---

### 7.2 CosyVoice 3 Dialect Chain
**Goal:** Test how far dialect variation can be pushed within a single model.

**Setup:**
- Generate the same sentence in: Standard Mandarin, Cantonese, Sichuan dialect, Shanghainese
- Have a native speaker of each variant rate intelligibility (1–5) and accent authenticity (1–5)

**Measure:** Intelligibility and authenticity scores per dialect

**What you'd learn:** Whether CosyVoice 3's 18+ dialect support is production-quality for each dialect, or whether some dialects are significantly weaker than others.

---

### 7.3 Voice Identity Interpolation
**Goal:** Map what happens to voice identity as you blend between two reference speakers.

**Setup:**
- Clone voice A (Speaker 1) and voice B (Speaker 2) with Chatterbox
- Generate the same sentence with: 100% A, 80/20 blend, 60/40, 50/50, 40/60, 20/80, 100% B — if the model supports reference blending; otherwise use progressive mixing of reference clips
- Score speaker similarity to A and B at each blend point

**Measure:** Speaker similarity curves for A and B as a function of blend ratio; the crossover point

**What you'd learn:** Whether voice identity degrades smoothly or snaps between attractors — relevant for building smoothly adjustable voice persona systems.

---

### 7.4 LLM Screenplay → Full Audio Drama (No Humans)
**Goal:** Build a complete multi-character audio drama using only AI, zero human voice recording.

**Setup:**
- Prompt Claude to write a 5-minute, 3-character screenplay (with scene direction, not just dialogue)
- Assign: characters A and B → Dia-2B (S1/S2), character C → Chatterbox with a cloned reference voice
- Generate all dialogue segments, stitch in sequence with silence gaps per scene direction
- Add ambient sound via Ming-Omni for key scenes

**Measure:** End-to-end production time; subjective coherence rating from a listener who doesn't know it's AI-generated

**What you'd learn:** The current ceiling of fully automated AI audio production — and exactly where the remaining human bottlenecks are (script review, voice matching, segment stitching, quality control).

---

## Priority Order

| Priority | Experiment | Effort | Payoff |
| --- | --- | --- | --- |
| High | 3.1 Round-trip latency benchmark | Low | Directly informs any voice agent build |
| High | 2.1 Emotion paradigm shootout | Low | Answers which system to use for expressive TTS |
| High | 5.1 UTMOS scoring | Medium | Makes all subjective ratings reproducible |
| High | 3.5 Voice-to-voice translation | Medium | Tests the most exciting CosyVoice 3 capability |
| Medium | 1.2 Voice consistency (30 lines) | Low | Validates Chatterbox for audiobook production |
| Medium | 3.3 Emotion-aware pipeline | Medium | Tests Kimi-Audio SER in a real pipeline |
| Medium | 4.1 Watermark stress test | Medium | Establishes real-world robustness of Chatterbox |
| Medium | 2.3 LLM-directed emotion | Low | Validates fully automated emotion direction |
| Lower | 6.1 Dialogue dataset factory | High | High value but long-running |
| Lower | 7.4 Full audio drama | High | Creative showcase but not immediately practical |
