# Benchmark and Evaluation Details

This document summarizes the current manuscript, **ContextSpoof: A Mandarin Benchmark for Temporal Localization of LLM-Guided Speech Manipulation**. The numbers below are reported in that manuscript; they have not been recomputed from this repository's four review pairs. This repository remains a qualitative companion artifact, not a complete reproducibility release.

## Construction and Partitioning

AISHELL-3 supplies 85 hours of source recordings from 218 speakers. Source clips lasting 1.5–8.0 seconds are retained, converted to 16-kHz mono WAV, and normalized to −23 LUFS. Construction comprises three stages:

1. **Content manipulation:** Qwen-3.5-Max generates Negation Tampering, Entity Replacement, Quantity Tampering, or Detail Injection edits. Structured outputs retain the original and modified transcripts, strategy, and changed text span; malformed responses are rejected.
2. **Zero-shot synthesis:** IndexTTS, CosyVoice, F5-TTS, or Spark-TTS renders the modified segments.
3. **Reintegration and labels:** Montreal Forced Aligner supplies construction-time alignment. Synthesized segments are inserted using Zero-Crossing Cut, Crossfade, Overlap-Add, or Phase-Aligned Crossfade. Inserted intervals provide temporal reference labels; failed construction attempts are regenerated or excluded.

The benchmark contains 170,848 clips, approximately 160 hours, with equal numbers of bona fide and partially spoofed recordings. Training, validation, and test contain 74,718, 49,812, and 46,318 clips, respectively. All derivatives of a source utterance are assigned to the same partition, with zero pairwise overlap of source-utterance identifiers.

Training uses approximately 20% Zero-Crossing Cut, 40% Crossfade, and 40% Overlap-Add. Phase-Aligned Crossfade occurs only in the test set, comprising 5,771 partially spoofed clips. All four TTS systems occur in training; the held-out construction condition is a **reintegration method**, not an unseen TTS system.

The test set contains 19,622 seen-speaker clips and 26,696 unseen-speaker clips. Seen speakers are represented in training, but their test utterances are new. Unseen speakers occur in neither training nor validation. Of the test clips, 23,159 are partially spoofed: 9,811 from seen speakers and 13,348 from unseen speakers. A clip can contain several disjoint reference events. The [README partition table](../README.md#current-benchmark-snapshot) includes the class counts.

## Localization Method

The text-independent model uses EAT features, Conformer temporal encoding, predicted boundary gating, bidirectional LSTM aggregation, and a linear-chain CRF. Its positional labels are:

| Label | Meaning |
| --- | --- |
| B | Bona fide background |
| O | Onset of a manipulated segment |
| I | Interior of a manipulated segment |
| E | Offset of a manipulated segment |

Here **B means background and O means onset**; they should not be interpreted using a different tagging convention. Transcripts are not inference inputs.

The training objective combines CRF negative log-likelihood with frame-level cross-entropy weighted by distance to the nearest reference boundary. Boundary gating uses predicted boundary probabilities to modulate features, whereas boundary weighting uses reference positions to allocate training emphasis. Cross-Segment Mixing recombines audio portions and their positional labels together, and is applied only to training samples. Validation and test waveforms and labels are unchanged by this augmentation.

The manuscript specifies PyTorch, AdamW, 50 epochs, batch size 32, backbone/downstream learning rates of 1e-5/1e-4, loss balance λ = 0.3, boundary emphasis α = 1.5, and boundary spread σ = 3 frames. These reported settings are not a substitute for the implementation and full configurations planned for release.

## Evaluation Protocol

All compared systems are retrained on the ContextSpoof training/validation partitions. PSD-L, IFBDN, and CFPRF use their authors' public implementations. RawNet2 and AASIST are adapted for frame localization by removing global pooling. Model selection, early stopping, and thresholds use training/validation data only; the test set is reserved for final evaluation.

| Measure | Reported evaluation scope | Definition or condition |
| --- | --- | --- |
| Clip-level EER | Balanced full test set, including bona fide and partially spoofed clips | Utterance score is the maximum frame-level spoof probability |
| Event-P / Event-R / Event-F1 | Partially spoofed test clips | Both predicted onset and offset must fall within 200 ms of the corresponding reference boundaries under the DCASE-style event protocol |
| Segment-F1 | Partially spoofed test clips | Overlap criterion is strictly IoU > 0.5; this is distinct from the event boundary-tolerance criterion |
| Frame-F1 | Partially spoofed test clips | Frame-level bona fide/manipulated classification |
| Boundary MAE | Paired reference events in partially spoofed test clips | Mean absolute onset and offset error over the selected pairs |
| Short-event MAE | Paired reference events whose reference duration is < 300 ms | The same boundary-error calculation, restricted by reference duration |

For boundary-error analysis, each reference event independently selects its highest-IoU prediction whenever a prediction is present. A prediction can therefore be selected by multiple reference events. This pairing does not impose the Event-F1 tolerance or Segment-F1 overlap threshold and must not be interpreted as one-to-one event detection matching. Pair counts are event counts, not clip counts or true-positive counts. MAE is conditional on available pairs, and different variants can have different paired-event populations.

The main baseline comparison reports **mean ± standard deviation over three random initializations for every method**. Condition-wise results, ablations, boundary-error analysis, and the loss-weight sweep use one fixed seed across their compared settings; their values are not three-run means. Small mean differences alone do not establish statistical significance.

## Main Results

All entries are percentages. Clip EER uses the full test set; the five localization columns use the partially spoofed subset. Lower EER and higher localization scores are better.

| Model | Clip EER ↓ | Event-F1 ↑ | Event-P ↑ | Event-R ↑ | Segment-F1 ↑ | Frame-F1 ↑ |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| RawNet2 | 12.10 ± 0.57 | 34.1 ± 0.4 | 35.2 ± 0.3 | 33.0 ± 0.4 | 34.1 ± 0.5 | 83.5 ± 0.2 |
| AASIST | 8.26 ± 0.52 | 38.1 ± 0.3 | 35.8 ± 0.2 | 40.6 ± 0.3 | 38.1 ± 0.6 | 84.6 ± 0.1 |
| PSD-L | 5.23 ± 0.41 | 92.9 ± 0.2 | 92.8 ± 0.1 | 93.1 ± 0.2 | 90.6 ± 0.2 | 89.8 ± 0.1 |
| IFBDN | 3.81 ± 0.36 | 95.1 ± 0.2 | 95.6 ± 0.2 | 94.6 ± 0.2 | 91.9 ± 0.3 | 91.6 ± 0.2 |
| CFPRF | 2.09 ± 0.25 | 96.1 ± 0.2 | 95.8 ± 0.1 | 96.4 ± 0.2 | 93.3 ± 0.3 | 93.1 ± 0.1 |
| **Ours** | **1.99 ± 0.23** | **97.2 ± 0.1** | **97.5 ± 0.2** | **96.9 ± 0.1** | **95.7 ± 0.1** | **94.8 ± 0.1** |

The proposed method has the best reported mean on each metric among these compared methods under the shared protocol. Relative to CFPRF, Event-F1 increases by 1.1 percentage points, Segment-F1 by 2.4 points, and Frame-F1 by 1.7 points; EER decreases by 0.10 points. This comparison is limited to ContextSpoof and the listed baselines.

## Condition-Wise Results

The following fixed-seed localization scores are percentages on partially spoofed test clips. These condition groups overlap: the same clip has a TTS system, a splicing method, and a tamper type.

| Factor | Condition | Event-F1 | Segment-F1 | Frame-F1 |
| --- | --- | ---: | ---: | ---: |
| TTS | CosyVoice | 97.6 | 97.4 | 96.7 |
| TTS | F5-TTS | 96.9 | 93.5 | 92.7 |
| TTS | Spark-TTS | 97.5 | 96.7 | 94.7 |
| TTS | IndexTTS | 97.0 | 95.2 | 94.5 |
| Splicing | Overlap-Add | 97.1 | 95.7 | 95.0 |
| Splicing | Crossfade | 97.2 | 95.8 | 95.0 |
| Splicing | Zero-Crossing Cut | 97.4 | 95.8 | 94.3 |
| Splicing | Phase-Aligned Crossfade (held out) | 97.2 | 95.5 | 94.9 |
| Tamper type | Negation Tampering | 97.3 | 95.4 | 94.2 |
| Tamper type | Entity Replacement | 96.9 | 95.8 | 95.1 |
| Tamper type | Quantity Tampering | 96.7 | 95.5 | 94.1 |
| Tamper type | Detail Injection | 98.0 | 97.6 | 97.3 |
| Speakers | Unseen speakers | 96.8 | 95.1 | 94.5 |
| Aggregate | Partial-spoof subset | 97.2 | 95.7 | 94.8 |

## Component and Boundary Analyses

The fixed-seed ablation results are percentages. `w/o Boundary` denotes the manuscript's boundary-weighted-supervision ablation, not removal of all boundary-related model features. No separate CSM ablation row is reported in the current manuscript table.

| Variant | Event-F1 | Segment-F1 | Frame-F1 |
| --- | ---: | ---: | ---: |
| Full model | 97.2 | 95.7 | 94.8 |
| w/o Boundary | 96.3 | 94.9 | 94.5 |
| HuBERT + Ours | 90.4 | 84.4 | 84.4 |
| WavLM + Ours | 94.2 | 90.7 | 90.1 |
| w/o Conformer | 95.1 | 95.4 | 94.5 |
| w/o BiLSTM | 96.7 | 95.6 | 94.7 |
| w/o CRF | 94.5 | 91.3 | 90.6 |

| Variant | Mean MAE (ms) ↓ | Short-event MAE (ms) ↓ | Paired reference events | Short-event pairs |
| --- | ---: | ---: | ---: | ---: |
| Full model | 24.85 | 32.40 | 26,432 | 13,561 |
| w/o Boundary | 28.35 | 34.65 | 24,162 | 13,028 |

Boundary weighting reduces reported MAE by 3.50 ms overall (12.3%) and 2.25 ms for short events (6.5%), with more paired events in both groups. The compared pair populations differ, as defined above. In the manuscript's loss-weight sweep, all three localization metrics peak at λ = 0.3. These analyses are reported results; neither the sweep data nor a runnable evaluator is bundled here.
