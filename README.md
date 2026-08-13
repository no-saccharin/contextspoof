# ContextSpoof

Anonymous submission to MMM 2027.

## Overview

ContextSpoof is a Mandarin benchmark for temporal localization of LLM-guided partial speech manipulation. It studies recordings in which a short semantic unit is edited, synthesized with zero-shot TTS, and reintegrated into an otherwise bona fide utterance. Construction-time labels specify the manipulated span.

The benchmark covers four manipulation types (negation, entity substitution, quantity manipulation, and detail injection), four zero-shot TTS systems, and four reintegration methods. Its paper describes the construction protocol, source-utterance-level split, and evaluation setup.

## Review Materials

This anonymous repository provides four illustrative real/manipulated audio pairs, waveform visualizations, and a machine-readable manifest in [`audio_samples/samples.json`](audio_samples/samples.json). These sample audio pairs were selected from the actual ContextSpoof dataset. The examples cover the four manipulation types and are intended only to illustrate the task. They are not an evaluation set and do not constitute the complete ContextSpoof dataset.

| Manipulation | Synthesis | Reintegration |
| --- | --- | --- |
| Quantity manipulation | F5-TTS | Crossfade |
| Detail injection | Spark-TTS | Overlap-add |
| Entity substitution | IndexTTS | Phase-aligned crossfade |
| Negation | CosyVoice | Zero-crossing cut |

## Availability

Code and the ContextSpoof dataset will be released publicly upon acceptance, subject to the licenses of third-party resources. The planned release includes split manifests, metadata, construction configurations, training and evaluation scripts, and reproduced baseline implementations.

## Repository Layout

```text
audio_samples/
  samples.json                  # Sample-pair manifest and construction-time spans
  *_Real.wav                    # Source audio previews
  *_<type>_<tts>_<splice>.wav   # Manipulated audio previews
  *_viz.png                     # Corresponding waveform visualizations
```

## Responsible Use

The materials are provided for research on multimedia verification and temporal localization. They should not be used to misrepresent a speaker or to facilitate deceptive audio editing.

## License

No license is granted for this review package. Licensing terms for released code, generated assets, and associated metadata will be provided with the public release.
