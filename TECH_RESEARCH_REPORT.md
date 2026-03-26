# Church Sermon Real-Time Translation App: Technology Research Report

**Date**: March 2026
**Scope**: STT, Translation, TTS, and end-to-end pipeline evaluation for Korean sermon translation to English and Chinese

---

## Table of Contents

1. [Real-Time STT (Speech-to-Text) APIs](#1-real-time-stt-speech-to-text-apis)
2. [Real-Time Translation APIs](#2-real-time-translation-apis)
3. [TTS (Text-to-Speech) APIs](#3-tts-text-to-speech-apis)
4. [End-to-End Real-Time Translation Solutions](#4-end-to-end-real-time-translation-solutions)
5. [Noise Cancellation / Speaker Isolation](#5-noise-cancellation--speaker-isolation)
6. [Architecture Considerations](#6-architecture-considerations)
7. [Cost Estimation](#7-cost-estimation-12-hoursmonth-2-languages)
8. [Recommendation](#8-recommendation)

---

## 1. Real-Time STT (Speech-to-Text) APIs

### Comparison Matrix

| Feature | OpenAI gpt-4o-transcribe | OpenAI Whisper API | Google Cloud STT v2 | Azure Speech Services | Deepgram Nova-3 | AssemblyAI Universal |
|---|---|---|---|---|---|---|
| **Korean Quality** | Good (100+ langs, improved WER) | Fair (limited Korean training data) | Good (native Korean support) | Good (Korean supported) | Fair (Tier 2, 7-16% WER) | Korean batch only, NO streaming |
| **Real-Time Streaming** | Via Realtime API only | NO (batch only) | YES | YES | YES (native WebSocket) | NO Korean streaming |
| **Latency** | ~1.5-2s per chunk | ~2-5s (batch) | ~300-500ms streaming | ~100-300ms streaming | ~300ms (P50) | ~300ms (English only) |
| **Noise Handling** | Good (GPT-4o multimodal) | Moderate | Good (enhanced models) | Good (built-in) | Excellent (trained on noisy audio) | Good |
| **Speaker Diarization** | Yes (with diarization model) | No | Yes | Yes | Yes | Yes |
| **Price/minute** | $0.006 | $0.006 | $0.024 (standard) / $0.036 (enhanced) | $0.017 (real-time) | $0.0077 (streaming) | $0.0045 |
| **Price/hour** | $0.36 | $0.36 | $1.44 / $2.16 | $1.02 | $0.462 | $0.27 |
| **Free Tier** | None | None | 60 min/month | 5 hrs/month | Limited | 100 hrs one-time |

### Key Findings - Korean STT

- **OpenAI Whisper** has known weaknesses with Korean due to limited Korean training data (only ~17% of 680K training hours was non-English, spread across 98 languages). The newer **gpt-4o-transcribe** model improves on this significantly.
- **Google Cloud STT v2** has strong native Korean support with dedicated Korean language models and real-time streaming capability.
- **Azure Speech Services** offers the lowest latency (100-300ms) for streaming with solid Korean support, plus a generous 5-hour/month free tier.
- **Deepgram Nova-3** classifies Korean as Tier 2 (7-16% WER), meaning lower accuracy than Tier 1 languages. However, it excels at handling noisy audio natively.
- **AssemblyAI** does NOT support Korean for real-time streaming as of early 2026 (only English, Spanish, French, German, Italian, Portuguese).

### STT Verdict for Korean Sermons

**Top picks**: Azure Speech Services (best latency + Korean quality) or Google Cloud STT v2 (strong Korean models). For budget-conscious use, OpenAI gpt-4o-mini-transcribe at $0.003/min is the cheapest option with reasonable Korean quality.

---

## 2. Real-Time Translation APIs

### Comparison Matrix

| Feature | Google Cloud Translation v3 | DeepL API | OpenAI GPT-4o | Azure Translator |
|---|---|---|---|---|
| **Korean -> English** | Good | Good (added late 2024) | Excellent (context-aware) | Good |
| **Korean -> Chinese (Simplified)** | Good | Supported | Good | Good |
| **Korean -> Chinese (Traditional)** | Good | Supported | Good | Good |
| **Latency** | ~100-300ms | ~200-500ms | ~500-1500ms (chat API) | ~100-300ms |
| **Religious/Biblical Context** | No special handling | No special handling | Excellent (promptable context) | No special handling |
| **Streaming Support** | No (request/response) | No (request/response) | Yes (streaming chat) | No (request/response) |
| **Price** | $20/M chars | $25/M chars + $5.49/mo base | ~$2.50/M input + $10/M output tokens | $10/M chars |
| **Free Tier** | 500K chars/month | 500K chars/month | None | 2M chars/month |

### Key Findings - Translation Quality

- **OpenAI GPT-4o** is the standout for religious context translation. You can include system prompts like "You are translating a Korean church sermon. Use standard biblical terminology (e.g., 은혜 -> grace, 구원 -> salvation, 성령 -> Holy Spirit)" to dramatically improve domain-specific accuracy. GPT-4o also handles Korean-to-English context-aware translation better than traditional NMT systems in research benchmarks.
- **DeepL** added Korean support in late 2024. Their next-gen LLM model supports Korean-to-English and Korean-to-Chinese, but quality for Asian language pairs is generally considered behind Google's.
- **Google Cloud Translation** is reliable across all three language pairs (Korean -> English, Simplified Chinese, Traditional Chinese) with the lowest latency for traditional NMT.
- **Azure Translator** is the cheapest at $10/M characters with excellent language coverage (100+ languages) and a generous free tier.

### Translation Approach for Sermons

**Recommended**: Use OpenAI GPT-4o (or GPT-4o-mini for cost savings) with a custom system prompt containing:
- Biblical/theological glossary (Korean -> English, Korean -> Chinese)
- Instruction to preserve sermon tone and register
- Context about the specific denomination's terminology preferences

This approach leverages LLM contextual understanding to produce far superior translations for religious content compared to traditional NMT APIs.

---

## 3. TTS (Text-to-Speech) APIs

### Comparison Matrix

| Feature | OpenAI gpt-4o-mini-tts | OpenAI tts-1 | Google Cloud TTS | Azure TTS | ElevenLabs |
|---|---|---|---|---|---|
| **English Voice Quality** | Excellent (steerable) | Good | Excellent (Chirp 3 HD) | Excellent (Neural HD V2) | Best-in-class |
| **Chinese Voice Quality** | Good | Good | Good (Chirp 3 HD Preview) | Good (140+ langs) | Good (Mandarin support) |
| **Latency** | ~200-500ms | ~500ms | ~200-400ms | ~200-400ms | ~75ms (Flash v2.5) |
| **Streaming** | Yes | Yes | Yes | Yes | Yes (WebSocket) |
| **Naturalness** | Very natural (steerable emotions) | Natural | Very natural | Very natural | Most natural |
| **Price/M chars** | ~$0.60 input + $12/M audio tokens | $15/M chars | $4-30/M chars | $15-30/M chars | ~$300/M chars (Creator plan) |
| **Price/hour of audio** | ~$0.90 | ~$15 (for ~1M chars) | $0.26-1.96 | $0.98-1.96 | ~$18 |
| **Free Tier** | None | None | 4M chars/month (standard) | 5M chars/month | 10K chars/month |

### Key Findings

- **ElevenLabs** has the best voice quality and lowest latency (75ms with Flash v2.5) but is by far the most expensive (~10-27x more than competitors). Not ideal for a cost-sensitive church use case.
- **OpenAI gpt-4o-mini-tts** offers excellent value with steerable voices (you can instruct tone and emotion), good multilingual support, and low cost. The "steerability" feature is uniquely useful for sermon translation -- you can instruct it to "speak in a warm, pastoral, reverent tone."
- **Google Cloud TTS** with standard voices offers the cheapest option at $4/M characters, with 4M free characters/month. The newer Chirp 3 HD voices at $30/M are higher quality.
- **Azure TTS** offers 500+ voices across 140+ languages with a generous 5M character/month free tier.

### TTS Verdict

**Best value**: Google Cloud TTS (standard/WaveNet voices) or OpenAI gpt-4o-mini-tts. For a church budget, Google's free tier (4M chars/month) may cover the entire use case for standard voices.

---

## 4. End-to-End Real-Time Translation Solutions

### Option A: OpenAI Realtime API (Speech-to-Speech)

OpenAI provides an official cookbook for "Multi-Language One-Way Translation with the Realtime API" -- exactly the church sermon use case.

**How it works**:
- Speaker app captures Korean sermon audio via microphone
- Audio streams to OpenAI Realtime API via WebSocket
- Model translates and generates speech in target language in real time
- Listener app plays translated audio

**Pricing (gpt-realtime)**:
- Audio input: $32/M tokens (~$0.02/min)
- Audio output: $64/M tokens (~$0.24/min for generated speech)
- Total: ~$0.26/min = ~$15.60/hr per target language

**Pricing (gpt-realtime-mini)**:
- Audio input: $10/M tokens
- Audio output: $20/M tokens
- Total: ~$0.08/min = ~$4.80/hr per target language

**Pros**:
- Single API call handles STT + Translation + TTS
- Lowest complexity architecture
- Natural-sounding output
- Supports Korean input
- OpenAI maintains the cookbook example specifically for this use case

**Cons**:
- Most expensive option per hour
- Turn-based nature requires brief pauses between segments
- Less control over individual pipeline stages
- Harder to customize biblical terminology (though system prompt helps)

### Option B: Cascaded Pipeline (STT -> Translation -> TTS)

Build separate stages: capture audio -> transcribe -> translate -> synthesize speech.

**Typical latency budget**:
| Stage | Latency |
|---|---|
| Audio capture + buffering | 500-1000ms |
| STT (streaming) | 300-500ms |
| Translation | 100-500ms |
| TTS (streaming) | 200-500ms |
| Network overhead | 100-200ms |
| **Total end-to-end** | **1.2-2.7 seconds** |

**Pros**:
- Mix and match best-in-class for each stage
- Much cheaper per hour
- Full control over terminology via translation prompt
- Can display transcript alongside audio

**Cons**:
- Higher architectural complexity
- Latency compounds across stages
- More failure points

### Option C: Azure Speech Translation (Integrated)

Azure offers an integrated speech translation service that combines STT + Translation in one call.

**Pros**: Lower latency than separate calls, Korean supported, simpler than full cascade
**Cons**: Still needs separate TTS, less control over translation quality

---

## 5. Noise Cancellation / Speaker Isolation

### The Problem
Church environments have echo, congregation noise, music, multiple speakers, and reverberant acoustics. Isolating the pastor's voice is critical for STT accuracy.

### Solutions

| Solution | Type | Latency | Quality | Cost | Platform |
|---|---|---|---|---|---|
| **Picovoice Koala** | On-device SDK | Real-time | Excellent (5x better than RNNoise) | Free (open-source-ish) / Commercial license | iOS, Android, Web, Desktop |
| **Krisp SDK** | On-device | Real-time | Excellent | Commercial license | Desktop, Mobile |
| **RNNoise** | On-device (open source) | Real-time | Good (dated) | Free (Mozilla Public License) | C library, any platform |
| **Deepgram native** | Cloud (built into STT) | N/A | Good | Included in STT price | Cloud only |
| **Azure Speech** | Cloud (built into STT) | N/A | Good | Included in STT price | Cloud only |
| **Hardware approach** | Wireless lavalier mic | N/A | Best | $50-200 one-time | Any |

### Recommended Approach

**Best solution: Hardware + Software combination**
1. **Wireless lavalier microphone** on the pastor (eliminates ~80% of ambient noise at the source). A Rode Wireless GO or similar ($100-200) dramatically improves all downstream accuracy.
2. **Picovoice Koala** as a preprocessing step on the client device before sending to STT API. It runs locally with no cloud dependency and is 5x more effective than RNNoise.
3. Many modern STT APIs (Deepgram Nova-3, Azure) are already trained on noisy audio and handle remaining noise well.

---

## 6. Architecture Considerations

### Recommended Architecture: Server-Side Processing with WebSocket Streaming

```
[Pastor's Mic]
    |
    v
[Client App (Mobile/Tablet)]
    | - Captures audio (WebRTC/MediaRecorder)
    | - Optional: Picovoice Koala noise suppression (on-device)
    | - Streams audio chunks via WebSocket
    v
[Server (Node.js / Python)]
    | - Receives audio stream
    | - Forwards to STT API (streaming WebSocket)
    | - Receives transcript chunks
    | - Sends text to Translation API (with biblical glossary prompt)
    | - Sends translated text to TTS API (streaming)
    | - Streams TTS audio back to listeners
    v
[Listener Apps (Mobile/Tablet/Web)]
    | - Receives translated audio stream via WebSocket
    | - Plays audio in real-time
    | - Optionally displays transcript
```

### Why Server-Side?

1. **Single STT stream**: Only one STT connection needed regardless of listener count. Korean audio is transcribed once, then translated to multiple languages.
2. **Cost control**: Server manages API keys and rate limiting.
3. **Caching**: Same translation serves all listeners of a given language.
4. **Simpler client**: Listeners only need to receive and play audio.

### Why NOT Client-Side?

- Each client would need its own API keys (security risk)
- N listeners = N duplicate STT calls (wasteful)
- Mobile devices have limited processing power for on-device models
- Battery drain on mobile devices

### Latency Optimization Strategies

1. **Sentence-level chunking**: Don't wait for long pauses. Split on sentence boundaries (~2-3 seconds of speech) to start translation early.
2. **Parallel processing**: As soon as STT returns a sentence, fire translation requests for all target languages simultaneously.
3. **Streaming TTS**: Use WebSocket-based TTS (ElevenLabs, OpenAI) to start playing audio before the entire sentence is synthesized.
4. **Edge server**: Deploy the server geographically close to the church (or use a nearby cloud region).
5. **Audio buffer management**: Maintain a small playback buffer (500ms) on the client to smooth out network jitter.
6. **WebSocket over REST**: Use persistent WebSocket connections for all API calls to eliminate connection overhead.

### Target End-to-End Latency

With optimized pipeline: **2-4 seconds** from pastor speaking to listener hearing translation. This is acceptable for sermon listening (not a conversation).

---

## 7. Cost Estimation (12 hours/month, 2 languages)

**Assumptions**:
- 3 sessions/week, ~1 hour each = ~12 hours/month
- 2 target languages: English, Chinese
- ~150 words/minute speaking rate
- ~750 characters/minute Korean text (~9,000 chars/hour)
- Audio: 12 hours input, 24 hours output (2 languages)

### Option 1: OpenAI Realtime API (End-to-End)

| Component | Calculation | Monthly Cost |
|---|---|---|
| gpt-realtime audio input | 12 hrs x $0.02/min x 60 | $14.40 |
| gpt-realtime audio output | 12 hrs x $0.24/min x 60 x 2 langs | $345.60 |
| **Total** | | **~$360/month** |

With gpt-realtime-mini:
| Component | Calculation | Monthly Cost |
|---|---|---|
| Mini audio input | 12 hrs x 60 x $0.007/min | $5.04 |
| Mini audio output | 12 hrs x 60 x $0.07/min x 2 langs | $100.80 |
| **Total** | | **~$106/month** |

### Option 2: Cascaded Pipeline (Budget-Optimized)

| Component | Service | Calculation | Monthly Cost |
|---|---|---|---|
| STT | Azure Speech (streaming) | 12 hrs x $1.02/hr | $12.24 |
| Translation | OpenAI GPT-4o-mini | ~108K chars x 2 langs x ~$0.30/M tokens | ~$1.50 |
| TTS | Google Cloud TTS (WaveNet) | ~216K chars x $16/M chars | ~$3.46 |
| **Total** | | | **~$17/month** |

### Option 3: Cascaded Pipeline (Quality-Optimized)

| Component | Service | Calculation | Monthly Cost |
|---|---|---|---|
| STT | Google Cloud STT v2 (enhanced) | 12 hrs x $2.16/hr | $25.92 |
| Translation | OpenAI GPT-4o | ~108K chars x 2 langs x ~$1.00/M tokens | ~$5.00 |
| TTS | OpenAI gpt-4o-mini-tts | ~24 hrs audio output x $0.90/hr | ~$21.60 |
| **Total** | | | **~$53/month** |

### Option 4: Cascaded Pipeline (Free Tier Maximized)

| Component | Service | Calculation | Monthly Cost |
|---|---|---|---|
| STT | Azure Speech | 5 hrs free, 7 hrs x $1.02 | $7.14 |
| Translation | Azure Translator | 2M chars free (covers all usage) | $0.00 |
| TTS | Google Cloud TTS (Standard) | 4M chars free (covers all usage) | $0.00 |
| **Total** | | | **~$7/month** |

*Note: Free tier option sacrifices some quality (standard vs neural voices, NMT vs LLM translation) but is extremely affordable.*

### Cost Summary

| Architecture | Monthly Cost | Quality | Complexity |
|---|---|---|---|
| OpenAI Realtime (full) | ~$360 | Excellent | Low |
| OpenAI Realtime (mini) | ~$106 | Very Good | Low |
| Cascade (quality) | ~$53 | Very Good | Medium |
| Cascade (budget) | ~$17 | Good | Medium |
| Cascade (free tiers) | ~$7 | Acceptable | Medium |

---

## 8. Recommendation

### Recommended Architecture: Cascaded Pipeline (Budget-Quality Balance)

For a church sermon translation app, the recommended approach balances cost, quality, and maintainability:

**STT**: Azure Speech Services (streaming)
- Best real-time latency (100-300ms)
- Good Korean language support
- 5 hrs/month free tier
- $1.02/hour after free tier

**Translation**: OpenAI GPT-4o-mini with custom biblical glossary prompt
- Best religious/biblical context handling via system prompt
- Streaming support for lower latency
- Extremely cheap for text translation (~$1-2/month)
- Custom glossary example:
  ```
  System: You are translating a Korean church sermon in real-time.
  Use standard biblical terminology: 은혜=grace, 구원=salvation,
  성령=Holy Spirit, 말씀=the Word, 십자가=the cross, 부활=resurrection,
  찬양=praise, 기도=prayer, 회개=repentance, 축복=blessing.
  Maintain a warm, reverent pastoral tone. Translate naturally,
  not word-for-word.
  ```

**TTS**: Google Cloud TTS (WaveNet/Neural2) or OpenAI gpt-4o-mini-tts
- Google: Cheapest with generous free tier, good English/Chinese voices
- OpenAI: Steerable tone ("speak warmly and clearly"), slightly higher cost

**Noise**: Wireless lavalier mic + Picovoice Koala preprocessing

**Estimated monthly cost**: $17-53/month depending on voice quality preferences.

### Alternative: Quick MVP with OpenAI Realtime API

For a rapid prototype or if development resources are limited, use the OpenAI Realtime API with the official one-way translation cookbook. This eliminates the need to build and manage a multi-stage pipeline at the cost of ~$106/month (mini) to ~$360/month (full).

---

## Sources

### STT
- [OpenAI Transcribe & Whisper API Pricing (Mar 2026)](https://costgoat.com/pricing/openai-transcription)
- [OpenAI API Pricing](https://developers.openai.com/api/docs/pricing)
- [Google Cloud Speech-to-Text Pricing](https://cloud.google.com/speech-to-text/pricing)
- [Azure Speech Services Pricing](https://azure.microsoft.com/en-us/pricing/details/speech/)
- [Deepgram Pricing](https://deepgram.com/pricing)
- [Deepgram STT Pricing Breakdown 2025](https://deepgram.com/learn/speech-to-text-api-pricing-breakdown-2025)
- [AssemblyAI Pricing](https://www.assemblyai.com/pricing)
- [AssemblyAI Multilingual Universal Streaming](https://www.assemblyai.com/blog/introducing-multilingual-universal-streaming)
- [Conquering Korean ASR with Low-bit Whisper](https://www.edge-ai-vision.com/2025/11/small-models-big-heat-conquering-korean-asr-with-low-bit-whisper/)
- [Best Speech to Text APIs 2025 (Pricing per Minute)](https://vocafuse.com/blog/best-speech-to-text-api-comparison-2025/)

### Translation
- [Google Cloud Translation Pricing](https://cloud.google.com/translate/pricing)
- [DeepL API Plans](https://support.deepl.com/hc/en-us/articles/360019925219-DeepL-API-plans)
- [DeepL Pro Available in South Korea](https://www.deepl.com/en/blog/deepl-pro-available-in-south-korea)
- [Azure Translator Pricing](https://azure.microsoft.com/en-us/pricing/details/translator/)
- [Translation API Pricing Comparison (Feb 2026)](https://www.buildmvpfast.com/api-costs/translation)
- [Best Translation API 2026: Google vs DeepL vs GPT-4](https://intlpull.com/blog/best-translation-api-2026)
- [ChatGPT Academic Translation for Religious Texts (2025)](https://journals.sagepub.com/doi/10.1177/21582440251343954)

### TTS
- [OpenAI TTS API Pricing Calculator (Mar 2026)](https://costgoat.com/pricing/openai-tts)
- [OpenAI Text to Speech Guide](https://developers.openai.com/api/docs/guides/text-to-speech)
- [Google Cloud Text-to-Speech Pricing](https://cloud.google.com/text-to-speech/pricing)
- [Azure Speech Services Pricing](https://azure.microsoft.com/en-us/pricing/details/cognitive-services/speech-services/)
- [ElevenLabs API Pricing](https://elevenlabs.io/pricing/api)
- [Best TTS APIs in 2026](https://www.speechmatics.com/company/articles-and-news/best-tts-apis-in-2025-top-12-text-to-speech-services-for-developers)
- [Best TTS APIs for Real-Time Voice Agents (2026 Benchmarks)](https://inworld.ai/resources/best-voice-ai-tts-apis-for-real-time-voice-agents-2026-benchmarks)

### End-to-End / Realtime
- [OpenAI One-Way Translation Cookbook](https://developers.openai.com/cookbook/examples/voice_solutions/one_way_translation_using_realtime_api)
- [Introducing gpt-realtime](https://openai.com/index/introducing-gpt-realtime/)
- [OpenAI Realtime API Pricing 2025 Calculator](https://skywork.ai/blog/agent/openai-realtime-api-pricing-2025-cost-calculator/)
- [GPT Realtime Mini Pricing Breakdown](https://www.eesel.ai/blog/gpt-realtime-mini-pricing)

### Noise Cancellation
- [Picovoice Koala Noise Suppression](https://picovoice.ai/platform/koala/)
- [Voice Isolator Guide 2025](https://picovoice.ai/blog/voice-isolator/)
- [Krisp AI Noise Cancellation](https://krisp.ai/)
- [Krisp for Church Tech](https://churchtechtoday.com/ai-tool-krisp/)
- [Deepgram: Noise Reduction Paradox in STT](https://deepgram.com/learn/the-noise-reduction-paradox-why-it-may-hurt-speech-to-text-accuracy)

### Architecture
- [Optimizing Latency in Real-Time Speech Translation Pipelines](https://www.weblineglobal.com/blog/optimizing-real-time-speech-translation-latency/)
- [Real-Time Speech-to-Speech Translation (Google Research)](https://research.google/blog/real-time-speech-to-speech-translation/)
- [Real-Time vs Turn-Based Voice Agent Architecture](https://softcery.com/lab/ai-voice-agents-real-time-vs-turn-based-tts-stt-architecture)
- [Real-Time TTS API for Low-Latency Speech Streaming (2026)](https://www.camb.ai/blog-post/real-time-tts-api-for-low-latency-speech-streaming)
