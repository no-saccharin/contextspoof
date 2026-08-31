# Sample Metadata

[`audio_samples/samples.json`](../audio_samples/samples.json) describes four qualitative review pairs. Schema 1.1 adds display labels and source-preview audio properties while preserving existing sample IDs, file paths, enum values, transcripts, construction timestamps, and manipulated-audio metadata. Readers of schema 1.0 fields can continue using those fields; consumers that require an exact version string should allow 1.1 before reading the additions.

## Fields and Interpretation

| Field | Meaning |
| --- | --- |
| `id` | Stable example identifier; not a dataset split or speaker label |
| `source_utterance_id` | Source utterance identifier linking the two recordings |
| `real_audio`, `manipulated_audio`, `visualization` | File paths relative to the directory containing `samples.json` |
| `manipulation_type` | Existing machine-readable edit-category value; see the display mapping below |
| `manipulation_label` | Display name aligned with the current manuscript |
| `original_transcript`, `modified_transcript` | The source and edited text supplied for qualitative inspection |
| `changed_spans` | List of construction-time edits; a list supports multiple disjoint spans even though each bundled example has one |
| `changed_spans[].operation` | `replace` or `insert`; insertions have an empty `original_text` |
| `changed_spans[].original_text`, `modified_text` | Text removed/replaced and the inserted content |
| `changed_spans[].start_ms`, `end_ms` | Construction-time manipulated-span boundaries in milliseconds on the manipulated recording's timeline; not model predictions |
| `tts_system` | Synthesis system used for the example |
| `reintegration_method` | Existing machine-readable splicing-method value |
| `real_duration_ms`, `real_sample_rate_hz` | Source-preview WAV duration, rounded to milliseconds, and actual sample rate |
| `manipulated_duration_ms`, `manipulated_sample_rate_hz` | Full manipulated WAV duration, rounded to milliseconds, and actual sample rate; duration is not the length of its manipulated span |

The metadata records boundaries in milliseconds, not frame indices. For a span, its duration is `end_ms - start_ms`. To locate it in the manipulated waveform, convert using that file's sample rate. Millisecond durations are rounded and need not equal the WAV's exact sample-derived duration; use the WAV header when exact sample counts are needed. This package does not specify a frame-label rasterization or interval-endpoint convention for training.

## Terminology Compatibility

The current manuscript standardizes the display names without requiring existing filenames or JSON category keys to change.

| Existing `manipulation_type` | Current manuscript display name |
| --- | --- |
| `negation` | Negation Tampering |
| `entity_substitution` | Entity Replacement |
| `quantity_manipulation` | Quantity Tampering |
| `detail_injection` | Detail Injection |

| Existing `reintegration_method` | Current manuscript display name |
| --- | --- |
| `zero_crossing_cut` | Zero-Crossing Cut |
| `crossfade` | Crossfade |
| `overlap_add` | Overlap-Add |
| `phase_aligned_crossfade` | Phase-Aligned Crossfade |

## Preview Audio Versus Benchmark Preprocessing

All four source previews are **44.1-kHz mono**, whereas all four manipulated previews are **16-kHz mono**. The files are retained unchanged. The paper's 16-kHz preprocessing describes the benchmark construction workflow; it does not imply that these packaged source previews have been resampled. No loudness-normalization assertion is made for the preview files. The two recordings can also differ in duration, so neither sample indices nor timestamps should be transferred between them without alignment.

The reference intervals and model predictions drawn in `*_viz.png` have different roles. Only the construction-time reference intervals are recorded in `changed_spans`; predicted intervals, checkpoint identity, seed, and visualization-generation code are not included. Existing images support qualitative inspection but cannot reproduce or certify the current aggregate results.

The manifest intentionally does not assign examples to training, validation, seen-speaker, or unseen-speaker subsets: those per-example assignments are not provided with this artifact. The Phase-Aligned Crossfade example illustrates a method held out in the benchmark protocol; its presence here does not make this four-pair package an evaluation set.
