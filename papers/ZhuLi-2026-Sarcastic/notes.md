# Modeling Sarcastic Speech: Semantic and Prosodic Cues in a Speech Synthesis Framework

- **Authors:** Zhu Li, Yuqing Zhang, Xiyuan Gao, Shekhar Nayak, Matt Coler (University of Groningen)
- **Venue/Year:** arXiv, 2025 (v3: Jun 2026)
- **Local PDF:** [Mdeling_Sarcastic_Speech.pdf](/home/vibhav/Documents/research-notes/papers)
- **Public link:** (https://arxiv.org/pdf/2510.07096v3)
- **Tags:** #nlp #speech-synthesis #pragmatics #sarcasm

## One-line summary
> Proposes a TTS framework that synthesizes sarcastic speech by combining LoRA-tuned LLM semantic embeddings with prosodic exemplars retrieved (RAG) from real sarcastic speech, showing both cues improve perceived/detected sarcasm.

## Key claims / contributions
- First framework to *synthesize* (not just detect) sarcastic speech, jointly and independently controlling semantic and prosodic cues
- Novel use of RAG for retrieving **style/prosody references** (audio) instead of text/facts — addresses scarcity of annotated sarcastic speech data
- Semantic and prosodic cues are complementary rather than purely additive in shaping human-perceived sarcasm

## Linguistic Elements
- **Prosody**: patterns of pitch, stress, rhythm, pacing, intonation that shape how something is said — independent of literal word meaning
- **Text → Speech mapping**: semantics live in *text* (what's said/meant); prosody lives in *speech* (how it's delivered) — sarcasm requires modeling both, since sarcasm often has **no clear lexical markers** (same sentence can be sincere or sarcastic depending purely on delivery)

## Method
- **Semantic encoding**: LLaMA 3-8B fine-tuned via LoRA on News Headlines Sarcasm dataset (~28K headlines) → sarcasm-aware text embedding **E_s** (final-layer hidden states, per-token)
- **Prosody retrieval (RAG)**: index of sarcastic utterances from MUStARD++, each keyed by its own **E_s**; new input's **E_s** used as query → top-K (K=3) most similar clips retrieved via cosine similarity
- **Phoneme Encoder** (standard VITS component, not novel to this paper)
	- Text → grapheme-to-phoneme (G2P) conversion → phoneme sequence (e.g., "cat" → /k/ /æ/ /t/)
	- Phoneme sequence → neural encoder → **E_p ∈ ℝ^(T_p × d_p)**
- **Cross-Attention (Phonemes × Semantics)**
	- Q = E_p, K = V = E_s
	- Each phoneme computes relevance (softmax) over all semantic tokens → output = weighted blend of E_s, per phoneme position
- **WavLM**
	- Pre-trained, frozen speech model; captures general acoustic patterns (pitch, rhythm, speaker style) via self-supervision, no labels needed
	- Raw output: **T frames × d_w**-dim vectors (one vector per ~20ms window)
	- **Pooling**: averages all T frame-vectors → single fixed-length vector; smooths out fine timing detail, retains general style/register (used as prosody proxy)
- **Prosody encoding**: retrieved clips passed through frozen **WavLM**, pooled into fixed-length prosody vector **E_w**
- **Fusion**: phoneme embeddings **E_p** (query) cross-attend over **E_s** (key/value) → **H**; pooled, projected **E_w** summed and added to H → **Z**
- **Synthesis**: **Z** feeds into VITS ("Sarcasm-aware VITS") in place of standard phoneme-encoder output, decoded to waveform

$$
Z = H + \sum_{k=1}^{K} W_w E_{w_k}, \quad H = \text{softmax}\left(\frac{E_p W_q (E_s W_k)^\top}{\sqrt{d_k}}\right) E_s W_v
$$

- 6 conditions tested: Baseline, BERT, raw LLaMA 3, LLaMA 3-LoRA, RAG-only, LoRA+RAG (combined)

## Evaluation metrics (definitions)
- **MOS (Mean Opinion Score)**: human subjective rating, 1–5 Likert scale, averaged across listeners
- **MCD (Mel-Cepstral Distortion)**: objective metric measuring spectral/timbral difference between generated and reference audio via mel-cepstral coefficients; lower = closer to natural reference
- **Downstream sarcasm detection**: MUStARD++ speech-only classifier run on *synthesized* audio, predictions checked against original ground-truth labels; reported as P/R/F1 — measures whether generated speech still "reads" as sarcastic to an automatic detector

## Results
- Semantic embedding comparison (sarcasm text classification, MUStARD++): LoRA-tuned LLaMA 3 best (F1 72.5%) vs BERT (66.8%), raw LLaMA 3 (65.5%)
- Synthesis eval: Combined (LoRA+RAG) best downstream sarcasm-detection F1 (62.5%), near ground truth (62.3%); lowest MCD (9.8)
- Raw (untuned) LLaMA 3 embeddings *hurt* naturalness/sarcasm ratings (NMOS 2.0, SMOS 2.6) vs baseline
- Human eval (30 listeners, NMOS/SMOS): Combined SMOS (3.8) ≈ LoRA-only SMOS (3.8) — prosody boosts objective F1 but not subjective SMOS beyond semantic-only

## Critique / open questions
- No explicit training objective given for new cross-attention/fusion layers; unclear if VITS is fine-tuned on sarcastic audio specifically
- Evaluation limited to isolated single sentences, no discourse-level context
- Minimal demographic data collected from human raters (only English proficiency, hearing status)

## How this connects to my work
- Relevant to NLP × pragmatics/formal semantics interest — models sarcasm as interaction of semantic incongruity + delivery, not lexical classification alone
- RAG-for-style-retrieval pattern (rather than fact retrieval) could generalize to other pragmatic phenomena for future research direction
