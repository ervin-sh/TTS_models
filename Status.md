# TTS Model Comparison — vs AWS Nova Sonic v2

Personal experiments testing real-time and open-source TTS models.
Data sourced from HuggingFace model pages, GitHub repos, and hands-on testing.

---

## Table A — Capabilities

| Model | Size | Latency | Streaming / Full-Duplex | Mac Compatible | Voice Cloning | Emotions | Watermarking | Semantic Extraction | License |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| [AWS Nova Sonic v2](https://aws.amazon.com/ai/generative-ai/nova/sonic/) | Not disclosed | Ultra-low (streaming) | **Yes – full-duplex** | Cloud only | Doesn't say | **Yes** | **Yes** | Yes (claimed, no clear verification) | Proprietary |
| [NVIDIA PersonaPlex](https://huggingface.co/nvidia/personaplex-7b-v1) | 7B | **170ms** (turn-taking) / 240ms (interruption) | **Yes – full-duplex** concurrent listen/speak | Linux + NVIDIA GPU only | **Yes** (voice prompt control, 0.650 SSIM) | No | Doesn't say | Yes | NVIDIA Open Model License |
| [VibeVoice-Realtime-0.5B](https://huggingface.co/microsoft/VibeVoice-Realtime-0.5B) | 0.5B | **~300ms** | Yes – streaming text-in / audio-out | **Yes** | No (single speaker only) | No | **Yes** (imperceptible + audible AI disclaimer) | Doesn't say | MIT |
| [Chatterbox](https://huggingface.co/ResembleAI/chatterbox) | 0.5B | **<200ms** | Not explicitly stated | **Yes** | **Yes** (zero-shot voice conversion) | **Yes** (emotion exaggeration control) | **Yes** (Perth watermarking — survives MP3 compression) | Doesn't say | **MIT** |
| [CosyVoice 3](https://huggingface.co/FunAudioLLM/Fun-CosyVoice3-0.5B-2512) | 0.5B | **~150ms** | **Yes – bi-streaming** (text-in & audio-out) | Not yet implemented — too tedious | **Yes** (zero-shot + cross-lingual) | Yes (instruct-based, e.g. `[laugh]`) | Doesn't say | Doesn't say | Apache 2.0 |
| [Dia-2B](https://huggingface.co/nari-labs/Dia2-1B) | 1B or 2B (two variants) | Can start with first few words | Yes – starts generating without full text | **Yes** | Yes (conditional via speaker prefixes) | Limited (speaker tags `[S1]` `[S2]` only) | No | No | Apache 2.0 |
| [DiVA](https://huggingface.co/WillHeld/DiVA-llama-3-v0-8b) | 8B | Not specified | No | Yes — very difficult, took too long | No | No | No | No | Apache 2.0 |
| [fishspeech S1 mini](https://huggingface.co/fishaudio/s1-mini) | 0.5B (mini) / 4B (full) | Very slow on Mac (designed for CUDA) | No | Yes — extremely annoying to set up | **Yes** | **Yes** (49 emotion + tone markers) | No | No | CC-BY-NC-SA-4.0 (**non-commercial only**) |
| [Kimi-Audio-7B-Instruct](https://huggingface.co/moonshotai/Kimi-Audio-7B-Instruct) | 7B | Not specified | Yes (chunk-wise streaming detokenizer) | No – CUDA only | No | Yes (SER — speech emotion recognition) | No | **Yes** (ASR, AQA, AAC, SER, SEC/ASC) | MIT / Apache 2.0 |
| [Kokoro-82M TTS](https://huggingface.co/hexgrad/Kokoro-82M) | 82M | Blazing fast locally (no official claim) | No | **Yes** | No (explicitly excluded) | No | No | No | **Apache 2.0** |
| [Magpie TTS 357M](https://huggingface.co/nvidia/magpie_tts_multilingual_357m) | 357M | Extremely fast (no official claim) | Yes (designed for streaming agents, 20s chunks) | **Yes** | No (zero-shot explicitly removed) | No (enterprise NIM only, not open model) | No | No | NVIDIA Open Model License |
| [Ming-Omni-TTS-0.5B](https://huggingface.co/inclusionAI/Ming-omni-tts-0.5B) | 0.5B | 3.1Hz (Patch-by-Patch compression) | Not stated | **Yes** | **Yes** (zero-shot) | **Yes** (fine-grained, 76.7% accuracy) | No | No | Apache 2.0 |
| [CSM-1B](https://huggingface.co/sesame/csm-1b) | 1B | Slow | Not stated | **Yes** | Limited (base model — requires fine-tuning for specific voices) | No | No | No | Apache 2.0 |
| [Maya1](https://huggingface.co/maya-research/maya1) | 3B | **Sub-100ms** (with vLLM) | **Yes – streaming** | **Yes (MPS/CPU)** | No | **Yes** (20+ inline tags `<laugh>` `<whisper>`) | No | No | **Apache 2.0** |

---

## Table B — Details, Features & Personal Notes

| Model | Language Support | Notable Features | Personal Audio Notes |
| --- | --- | --- | --- |
| [AWS Nova Sonic v2](https://aws.amazon.com/ai/generative-ai/nova/sonic/) | Multilingual | Prompt-injectable emotions and filler words e.g. `[laugh] That's funny` | Very natural |
| [NVIDIA PersonaPlex](https://huggingface.co/nvidia/personaplex-7b-v1) | English only (24kHz) | Dual-stream architecture; 90.8% smooth turn-taking success rate; text prompt control for persona attributes | N/A — not tested locally |
| [VibeVoice-Realtime-0.5B](https://huggingface.co/microsoft/VibeVoice-Realtime-0.5B) | English (primary) + 9 experimental: German, French, Italian, Japanese, Korean, Dutch, Polish, Portuguese, Spanish | Acoustic tokenizer with 3200× downsampling; diffusion-based decoding; long-form speech up to 10 min; multi-speaker TTS variant supports 90-min generation (full podcast) | Surprisingly natural — almost the same feel as Nova Sonic 2 |
| [Chatterbox](https://huggingface.co/ResembleAI/chatterbox) | Arabic, Danish, German, Greek, English, Spanish, Finnish, French, Hebrew, Hindi, Italian, Japanese, Korean, Malay, Dutch, Norwegian, Polish, Portuguese, Russian, Swedish, Swahili, Turkish, Chinese (23 languages) | 0.5B Llama backbone; alignment-informed inference; trained on 0.5M hours of cleaned data; first open-source TTS with emotion exaggeration control; CFG/Pace control | Very natural when you tune the correct exaggeration and weights |
| [CosyVoice 3](https://huggingface.co/FunAudioLLM/Fun-CosyVoice3-0.5B-2512) | Chinese, English, Japanese, Korean, German, Spanish, French, Italian, Russian + 18+ Chinese dialects (Guangdong, Minnan, Sichuan, Dongbei, Shan3xi, Shan1xi, Shanghai, Tianjin, Shandong, Ningxia, Gansu, etc.) | Pronunciation inpainting (Chinese Pinyin + English CMU); multi-dialect support; 77.4–78% speaker similarity; 1.68% WER | Not yet tested — implementation too tedious |
| [Dia-2B](https://huggingface.co/nari-labs/Dia2-1B) | English only (max 2 min audio) | Two-persona dialogue model with `[S1]` / `[S2]` speaker tags; word-level timestamps; Kyutai Mimi codec | Multiple persona dialogue sounds quite natural |
| [DiVA](https://huggingface.co/WillHeld/DiVA-llama-3-v0-8b) | Multiple (CommonVoice dataset, mostly English in practice) | End-to-end voice assistant (speech + text); distillation-based training; multimodal input; based on Llama-3.1-8B | Ugly, robotic |
| [fishspeech S1 mini](https://huggingface.co/fishaudio/s1-mini) | English, Chinese, Japanese, German, French, Spanish, Korean, Arabic, Russian, Dutch, Italian, Polish, Portuguese (13 languages) | 2M+ hours training; online RLHF; 49 emotion/tone markers: `(laughing)` `(sobbing)` `(whispering)` `(angry)` `(sad)` `(excited)` `(surprised)` `(satisfied)` `(delighted)` `(scared)` `(worried)` `(upset)` `(nervous)` `(frustrated)` `(depressed)` `(empathetic)` `(embarrassed)` `(disgusted)` `(moved)` `(proud)` `(relaxed)` `(grateful)` `(confident)` `(curious)` `(confused)` `(joyful)` etc. | Sounds funny |
| [Kimi-Audio-7B-Instruct](https://huggingface.co/moonshotai/Kimi-Audio-7B-Instruct) | English, Chinese (13M+ hours training data) | ASR, audio QA (AQA), audio captioning (AAC), speech emotion recognition (SER), sound event/scene classification (SEC/ASC), end-to-end speech conversation; hybrid acoustic + semantic tokens | Not tested |
| [Kokoro-82M TTS](https://huggingface.co/hexgrad/Kokoro-82M) | 🇺🇸 American English (11F 9M), 🇬🇧 British English (4F 4M), 🇯🇵 Japanese (4F 1M), 🇨🇳 Mandarin Chinese (4F 4M), 🇪🇸 Spanish (1F 2M), 🇫🇷 French (1F), 🇮🇳 Hindi (2F 2M), 🇮🇹 Italian (1F 1M), 🇧🇷 Brazilian Portuguese (1F 2M) — 54 voices total | Extremely small (82M) but natural-sounding; StyleTTS 2 + ISTFTNet vocoder; trained on ~$1,000 of compute | Very natural — surprising |
| [Magpie TTS 357M](https://huggingface.co/nvidia/magpie_tts_multilingual_357m) | English, Spanish, German, French, Vietnamese, Italian, Mandarin Chinese, Hindi, Japanese (9 languages) | 5 fixed speakers (Sofia, Aria, Jason, Leo, John Van Stan); multi-codebook prediction; local transformer refinement; trained on 50K hours; built-in text normalization; designed for streaming voice agents | Very natural — surprising |
| [Ming-Omni-TTS-0.5B](https://huggingface.co/inclusionAI/Ming-omni-tts-0.5B) | Chinese (primary), English, Cantonese | Industry-first unified generation of speech + ambient sound + music in single channel; handles math expressions and chemical equations; vocal controls: rate, pitch, volume, emotion, dialect; 0.83% WER (Chinese eval) | Ugly |
| [CSM-1B](https://huggingface.co/sesame/csm-1b) | English (primary; limited non-English due to data contamination) | Context-aware generation; multi-speaker dialogue; batch inference; CUDA graph compilation; fine-tuning compatible; native Transformers integration | Sounds great |
| [Maya1](https://huggingface.co/maya-research/maya1) | English (multi-accent) | Natural language zero-shot voice descriptions; 20+ emotion tags `<laugh>` `<whisper>` `<cry>` `<sigh>`; Llama-style 3B + SNAC codec 24kHz; vLLM production infra; prefix caching for repeated voices; runs on Mac via MPS/CPU | Very nice — ElevenLabs quality |

---

## Table C — Training Data & Evaluation Metrics

`■` = data not publicly disclosed

| Model | Training Dataset(s) | Dataset Size | Eval Benchmark(s) | Key Metrics |
| --- | --- | --- | --- | --- |
| [AWS Nova Sonic v2](https://aws.amazon.com/ai/generative-ai/nova/sonic/) | ■ | ■ | ■ | ■ |
| [NVIDIA PersonaPlex](https://huggingface.co/nvidia/personaplex-7b-v1) | Fisher English (LDC2004S13 + LDC2005S13) | ~7,303 conversations (up to 10 min each) | FullDuplexBench | Speaker Similarity (WavLM-TDNN): **0.650** · Turn-taking TOR: **90.8%** · Interruption TOR: **95.0%** · Interruption Latency: **240ms** |
| [VibeVoice-Realtime-0.5B](https://huggingface.co/microsoft/VibeVoice-Realtime-0.5B) | Not specified (base: Qwen2.5-0.5B) | ■ | LibriSpeech test-clean · SEED test-en | WER: **2.00%** · Spk Sim: **0.695** (LibriSpeech) · WER: **2.05%** · Spk Sim: **0.633** (SEED) |
| [Chatterbox](https://huggingface.co/ResembleAI/chatterbox) | Freely available internet audio | **0.5M hours** | ■ | ■ (no numerical metrics published) |
| [CosyVoice 3](https://huggingface.co/FunAudioLLM/Fun-CosyVoice3-0.5B-2512) | ■ | ■ | Internal CN/EN/Hard test sets | ZH CER: **1.21%** → **0.81%** (post RL) · ZH Spk Sim: **78.0%** · EN WER: **2.24%** → **1.68%** (post RL) · EN Spk Sim: **71.8%** · Hard CER: **6.71%** · Hard Spk Sim: **75.8%** |
| [Dia-2B](https://huggingface.co/nari-labs/Dia2-1B) | ■ | ■ | ■ | ■ |
| [DiVA](https://huggingface.co/WillHeld/DiVA-llama-3-v0-8b) | Mozilla CommonVoice 17.0 | ■ | ■ | Training only: 7,000 gradient steps · batch 512 · 11 hrs on TPU v4-256 |
| [fishspeech S1 mini](https://huggingface.co/fishaudio/s1-mini) | Internal proprietary corpus | **2M+ hours** (13 languages) | Internal | WER: **0.011** · CER: **0.005** · Speaker Distance: **0.380** |
| [Kimi-Audio-7B-Instruct](https://huggingface.co/moonshotai/Kimi-Audio-7B-Instruct) | ■ (diverse audio + text) | **13M hours** | ■ (see [technical report](https://raw.githubusercontent.com/MoonshotAI/Kimi-Audio/master/assets/kimia_report.pdf)) | ■ |
| [Kokoro-82M TTS](https://huggingface.co/hexgrad/Kokoro-82M) | Public domain audio · Apache/MIT licensed audio · synthetic TTS audio · Koniwa (CC BY 3.0) · SIWIS (CC BY 4.0) | **< few hundred hours** | See EVAL.md (not published on card) | ■ |
| [Magpie TTS 357M](https://huggingface.co/nvidia/magpie_tts_multilingual_357m) | Hi-FiTTS · HiFiTTS-2 · LibriTTS · CML-TTS · LSVSC · InfoRe-1/2 · AI4Bharat · Emilia YODAS · Common Voice + internal | **50,000 hours** (38K training) | LibriTTS test-clean · CML-TTS (ES/FR/DE) | CER: **0.34%** (EN) · **1.14%** (ES) · **2.70%** (FR) · **0.66%** (DE) · SV-SSIM: **0.835** (EN) · **0.715** (ES) · **0.703** (FR) · **0.626** (DE) |
| [Ming-Omni-TTS-0.5B](https://huggingface.co/inclusionAI/Ming-omni-tts-0.5B) | ■ | ■ | WSYue-TTS-Eval · WSC-TTS-Eval · CV3-Eval · Seed-TTS-Eval · InstructTTS-Eval-ZH | Dialect accuracy: **96%** (WSYue) · **86%** (WSC) · **93%** (Cantonese) · Emotion accuracy: **76.7%** (CV3 avg) · Voice clone WER: **0.83%** · Text norm CER: **1.97%** · Instruction following: **76.20%** · Inference rate: **3.1 Hz** |
| [CSM-1B](https://huggingface.co/sesame/csm-1b) | ■ | ■ | ■ | ■ |
| [Maya1](https://huggingface.co/maya-research/maya1) | Internet-scale English speech + proprietary curated studio recordings (human-verified, MFA-aligned, MinHash-LSH text deduped, Chromaprint audio deduped) | ■ | ■ | ■ |


---

## Quick Summary

| Criteria | Best Option(s) |
| --- | --- |
| Lowest latency | Maya1 (sub-100ms, vLLM), CosyVoice 3 (~150ms), Chatterbox (<200ms), PersonaPlex (170ms) |
| Full duplex | Nova Sonic v2, PersonaPlex |
| Best naturalness (personal) | Nova Sonic v2, Maya1, VibeVoice, Kokoro-82M, Magpie, CSM-1B |
| Most languages | Chatterbox (23), CosyVoice 3 (9 + 18 dialects), fishspeech (13) |
| Richest emotion control | fishspeech S1 mini (49 tags), Ming (fine-grained), Maya1 (20+ inline tags), Chatterbox (exaggeration) |
| Voice cloning | Chatterbox, CosyVoice 3, Ming, PersonaPlex |
| Watermarking | VibeVoice (dual), Chatterbox (Perth) |
| Lightest / fastest locally | Kokoro-82M (82M params), Magpie (357M), VibeVoice (0.5B) |
| Free for commercial use | Kokoro-82M, VibeVoice, CSM-1B, Chatterbox, Magpie, CosyVoice 3, Maya1 |
| **Non-commercial only** | fishspeech S1 mini (CC-BY-NC-SA-4.0) |

