# ContextSpoof

Anonymous submission to MMM 2027.

## Overview

ContextSpoof is a Mandarin benchmark for temporal localization of LLM-guided partial speech manipulation. It studies recordings in which one or more short semantic units are edited, synthesized with zero-shot TTS, and reintegrated into an otherwise bona fide utterance. Construction-time labels specify the manipulated spans.

The benchmark covers four manipulation types (negation, entity substitution, quantity manipulation, and detail injection), four zero-shot TTS systems, and four reintegration methods. Its paper describes the construction protocol, source-utterance-level split, and evaluation setup.

For event-level evaluation, a predicted manipulated span is counted as correct only when both its onset and offset are within 200 ms of the construction-time reference span. Segment-level evaluation additionally uses an IoU threshold of 0.5. For boundary-error analysis, each reference span independently selects the available predicted span of highest IoU; short events are reference spans shorter than 300 ms. Because one clip may contain several disjoint reference spans, event counts and clip counts are reported separately in the paper.

## Review Artifact

This repository is a compact companion artifact for double-blind review. It supports qualitative inspection of representative source and manipulated pairs and verification of the sample metadata format through four audio pairs, waveform visualizations, and a machine-readable manifest in [`audio_samples/samples.json`](audio_samples/samples.json). The examples were selected from ContextSpoof to cover all four manipulation types; they are reviewer-facing examples rather than evaluation samples.

Each waveform preview juxtaposes the source audio, the construction-time reference interval, and a model prediction generated for qualitative illustration. Aggregate evaluation results are reported in the paper.

| Manipulation | Synthesis | Reintegration |
| --- | --- | --- |
| Quantity manipulation | F5-TTS | Crossfade |
| Detail injection | Spark-TTS | Overlap-add |
| Entity substitution | IndexTTS | Phase-aligned crossfade |
| Negation | CosyVoice | Zero-crossing cut |

### Suggested Inspection

1. Compare each `*_Real.wav` file with its corresponding manipulated file.
2. Use `*_viz.png` to inspect the construction-time manipulated interval.
3. Check the transcript edit, timestamps, TTS system, and reintegration method in `audio_samples/samples.json`.

## Archival Release

Upon acceptance, we plan to publish the archival research package, including the ContextSpoof dataset, versioned prompts and model configurations, split manifests and metadata, training and evaluation code, and baseline implementations. Redistribution will follow the licenses governing the source corpus and third-party models.

## Repository Layout

```text
audio_samples/
  samples.json                  # Sample-pair manifest and construction-time spans
  *_Real.wav                    # Source audio previews
  *_<Type>_<TTS>_<Method>.wav   # Manipulated audio previews
  *_viz.png                     # Source/reference/prediction waveform previews
```

## Responsible Use

The materials are provided for research on multimedia verification and temporal localization. They should not be used to misrepresent a speaker or to facilitate deceptive audio editing.

## License

No license is granted for this review package. Licensing terms for released code, generated assets, and associated metadata will be provided with the public release.
