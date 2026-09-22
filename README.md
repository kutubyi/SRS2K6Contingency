# SRS2K6Contingency

Analysis code for a study of how social responsiveness (SRS-2) and psychological
distress (K6) relate to listener responses (nods and verbal backchannels) in online 
psychiatrist-led CBT-style interviews.

## Data preprocessing

### Recordings

Sixty-two adults completed a semi-structured online interview with the same
CBT-trained psychiatrist, who was blind to their questionnaire scores. Sessions
were recorded in Zoom's gallery view (1280 × 720 pixels, constant 25 frames per
second, H.264 video, AAC audio at 48 kHz). The layout was identical in every
recording: two 640 × 360-pixel tiles at the vertical centre of the frame, the
psychiatrist on the left and the participant on the right. Audio was a single
mixed track (the two stereo channels were identical), so speech was attributed
to speakers computationally (see *Speech activity*). For the first cohort
(sessions in December 2025 and January 2026, n = 36), the recordings had been
trimmed by 0.7–4.1 s at the start before transcription; these trimmed files were
analysed so that video, audio and transcripts share one clock. The remaining
recordings (n = 26) are unedited. Mean session length was 11.3 min (range
3.9–16.1). Each video was matched to its booked session slot using the
reservation records and, for unedited files, the recording timestamp.

### Questionnaires

The SRS-2 Adult Self-Report (65 items; Constantino & Gruber, 2012) was
administered online in Japanese. Responses (1 = not true to 4 = almost always
true) were scored 0–3, giving a raw total of 0–195, the five treatment
subscales (Social Awareness, 8 items; Social Cognition, 12; Social
Communication, 22; Social Motivation, 11; Restricted Interests and Repetitive
Behavior, 12) and Social Communication and Interaction (the first four combined).
Of the 17 items reverse-scored in the standard key, items 52 and 55 are worded
as difficulties rather than skills in the Japanese form administered here, and
both correlated positively with the rest of the scale (corrected item–total
r = .43 and .47 across all 107 identifiable questionnaire submissions); they
were therefore scored without reversal (15 reversed items). Totals under the standard 17-item key are reported as a sensitivity
check. Psychological distress was measured with the Japanese K6 (Kessler et al.,
2002; Furukawa et al., 2008; six items scored 0–4, total 0–24). Fifteen
participants submitted the questionnaire twice (typically at screening and
again before the session); the latest submission on or before the session day
was used.

### Face and head tracking

Each person's tile was cropped (ffmpeg; near-lossless H.264, CRF 12, frame
count and timing unchanged) and processed with OpenFace 2.2.0 (Baltrušaitis
et al., 2018) to obtain, per frame, head rotation (pitch, yaw, roll), facial
action unit intensities and gaze angles. Frames in which tracking failed or
OpenFace's confidence was below 0.8 were treated as missing. Gaps of up to
three frames (120 ms) were filled by linear interpolation; longer gaps were
left missing and flagged, so that no head movement was imputed across them.
Head pitch velocity (°/s) was computed by differentiating pitch after
zero-phase low-pass filtering (second-order Butterworth, 6 Hz cut-off) within
each unbroken stretch of tracking. Camera stability was checked by estimating
the frame-to-frame shift of the outer 15% of each tile (phase correlation at
5 Hz), since a moving camera shifts the background whereas head movement does
not. The psychiatrist's face was usable in a median of 99.9% of frames per
session (minimum 97.8%; median time in long gaps 0.5 s). The participant's face
was usable in a median of 99.7% of frames (minimum 93.9%; median time in long
gaps 1.5 s, maximum 29.5 s) in 61 of 62 sessions; in the remaining session the
face was tracked in 0.8% of frames (see *Sample and exclusions*). The
participant's camera moved during 5–20% of four sessions; these frames are
flagged.

### Speech activity and speaker attribution

Audio was resampled to 16 kHz mono and divided into 40-ms frames aligned with
the video frames. A frame was classed as containing sound when it fell within a
speech segment detected by Silero VAD (minimum speech 100 ms, minimum silence
100 ms) and its level exceeded −55 dBFS; Zoom's noise suppression lowers pauses
to about −80 dBFS, which makes short pauses between words visible.

Because both voices share one track, speech was attributed to speakers with a
classifier trained separately for each session, without manual labels.
Speaker embeddings (ECAPA-TDNN, SpeechBrain `spkrec-ecapa-voxceleb`;
Desplanques et al., 2020; Ravanelli et al., 2021) were computed on 0.8-s
windows every 0.2 s wherever the centre frame contained sound. Training labels
came from the video: mouth activity was quantified per person as the
frame-to-frame change in lip parting plus jaw drop (OpenFace AU25 + AU26),
averaged over 0.4 s, and windows in the top and bottom quarters of the log
ratio of the psychiatrist's to the participant's mouth activity were labelled
as psychiatrist and participant speech respectively. A logistic regression on
the embeddings, trained on these windows, gave a per-window probability that
the psychiatrist was speaking, which was averaged over all windows covering a
frame. Agreement between voice-based and face-based labels, assessed with
five-fold cross-validation, had a median of .978 across sessions (interquartile
range .960–.984, range .900–.997; 61 sessions, as the method requires both
faces). A
frame with sound was assigned to the psychiatrist when this probability was at
least .5 and to the participant otherwise; it was assigned to both (overlap)
when both people's mouth activity exceeded the 95th percentile of their own
activity during pauses.

To support the detection of brief backchannels overlapping the other person's
speech, the mixed audio was additionally separated into two voices with
SepFormer trained on WHAMR! (`speechbrain/sepformer-whamr16k`; Subakan et al.,
2021; Maciejewski et al., 2020), in 4-s chunks with 0.5 s of context on either
side. The two estimates of each chunk were rescaled by least squares to sum to
the mixture and assigned to the two speakers by the permutation whose
frame-wise amplitudes best matched those expected from the speaker
probabilities. The separated level of each voice per frame was retained as a
candidate signal; its use is determined by the validation against manual
annotation.

### Conversation phases

The interviews followed the opening of a CBT session (Beck, 2021; Ministry of
Health, Labour and Welfare, 2010): a mood check, a review of the past week
(including identifying the events that mattered most) and agenda setting,
preceded by a brief opening and followed by a closing. The psychiatrist opened
each phase with a recognizable question, and phase boundaries were located in
the session transcripts (automatic Whisper-based transcription with speaker
diarization, uncorrected) as the first segment containing the corresponding
question. To avoid matching a participant's mention of the same words, a match
required a question ending within the same or the following segment, and a
closing was accepted only from 20 s after the agenda question. Every proposed
boundary was then checked and corrected by a researcher
reading the transcript. [Number of corrected boundaries to be filled in after
review.]

### Sample and exclusions

Exclusion rules were fixed before questionnaire scores were linked to
behaviour. A session was included in analyses of participant behaviour if the
participant's face was tracked in at least 90% of the frames in which the
psychiatrist was speaking, the periods in which participants listen and
respond. Of the 62 sessions, 61 met this criterion (minimum 93.2%). In one
session the participant's camera faced a strong light source and the face was
tracked in 0.8% of frames; this session was excluded from all analyses of
participant behaviour, and speaker attribution, which requires both faces, was
not computed for it. In four sessions the participant's camera moved during
5–20% of the session; frames with camera movement are excluded from analyses
of head movement, but the sessions are retained. Participants with fewer than
20 listening opportunities for the primary cue are excluded at the analysis
stage.

### State tables

For each of the 61 sessions, all signals were combined into one table with a row per video
frame (25 Hz) on the original recording clock: both people's head rotation,
pitch velocity, facial action units and gaze, tracking confidence and gap
flags; sound, speaker attribution and separated-voice levels; the conversational
state (psychiatrist, participant, overlap or silence); and the conversation
phase. All downstream measures are computed from these tables.

### Software

Python 3.12 with NumPy 2.4.4, pandas 3.0.3, SciPy 1.17.1, scikit-learn 1.9.1,
OpenCV 4.13.0, PyTorch 2.5.1, SpeechBrain 1.1.1 and silero-vad 6.2.3; ffmpeg
7.1 (via imageio-ffmpeg 0.6.0); OpenFace 2.2.0 (Windows binaries).

### References

- Baltrušaitis, T., Zadeh, A., Lim, Y. C., & Morency, L.-P. (2018). OpenFace 2.0: Facial behavior analysis toolkit. *IEEE International Conference on Automatic Face & Gesture Recognition*, 59–66.
- Beck, J. S. (2021). *Cognitive behavior therapy: Basics and beyond* (3rd ed.). Guilford Press.
- Constantino, J. N., & Gruber, C. P. (2012). *Social Responsiveness Scale, Second Edition (SRS-2): Manual*. Western Psychological Services.
- Desplanques, B., Thienpondt, J., & Demuynck, K. (2020). ECAPA-TDNN: Emphasized channel attention, propagation and aggregation in TDNN based speaker verification. *Interspeech 2020*, 3830–3834.
- Furukawa, T. A., Kawakami, N., Saitoh, M., et al. (2008). The performance of the Japanese version of the K6 and K10 in the World Mental Health Survey Japan. *International Journal of Methods in Psychiatric Research, 17*(3), 152–158.
- Kessler, R. C., Andrews, G., Colpe, L. J., et al. (2002). Short screening scales to monitor population prevalences and trends in non-specific psychological distress. *Psychological Medicine, 32*(6), 959–976.
- Maciejewski, M., Wichern, G., McQuinn, E., & Le Roux, J. (2020). WHAMR!: Noisy and reverberant single-channel speech separation. *ICASSP 2020*, 696–700.
- Ministry of Health, Labour and Welfare (2010). うつ病の認知療法・認知行動療法 治療者用マニュアル.
- Ravanelli, M., Parcollet, T., Plantinga, P., et al. (2021). SpeechBrain: A general-purpose speech toolkit. *arXiv:2106.04624*.
- Silero Team (2024). Silero VAD: Pre-trained enterprise-grade voice activity detector. https://github.com/snakers4/silero-vad
- Subakan, C., Ravanelli, M., Cornell, S., Bronzi, M., & Zhong, J. (2021). Attention is all you need in speech separation. *ICASSP 2021*, 21–25.

## Running the preprocessing

```
python -m questionnaires.make_scores_table --valid-rule zoom_done   # questionnaire scores
python -m audit.video_inventory                                     # match videos to sessions
python -m features.run_openface --workers 3                         # face and head tracking
python -m features.speech_activity                                  # who speaks when
python -m features.separate_voices                                  # two-voice separation
python -m features.phase_boundaries                                 # proposed phase boundaries
python -m features.state_table                                      # 25 Hz state tables
python -m audit.tracking_qc                                         # tracking quality report
python -m unittest discover tests
```
