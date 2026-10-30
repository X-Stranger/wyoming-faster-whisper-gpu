# Wyoming Faster Whisper GPU + optional shared Jarvis encoder

Build with workflow_dispatch,upstream_tag=v3.8.1. The reviewed source patch is
pinned to upstream8c9142ecfa8fdb5b3cea82ad1dcad71cc24ebab9. Workflow tests the
extension without GPU/model downloads and publishes one Whisper image with unique
builder-commit tags. Original3.8.1-gpu-r1 tags are not overwritten.

Without --jarvis-encoder-uri the STT path keeps one worker and normal behavior.
With --jarvis-encoder-uri tcp://0.0.0.0:10500 and
--jarvis-encoder-model-directory <existing full-large-v3 snapshot directory>:
- one resident WhisperModel,num_workers=2,shared weights on one GPU;
- STT10300,encoder10500,one active STT plus one active encoder;
- bounded STT admission(one active+one waiting,3s queue wait);encoder never
  rejects solely because STT is active;an occupied encoder rejects new requests;
- cached weights and tested library versions checked before serving;
- unchanged Whisper language,VAD,name context,and decoding parameters;
- read-only profile embedded via explicit .dockerignore/package-data entries;
- health:python -m wyoming_faster_whisper.jarvis_encoder --check tcp://127.0.0.1:10500
  probes both endpoints without inference or loading another model.

CTranslate2 workers share weights but have extra working memory. Shared GPU
latency/VRAM under concurrent real STT+encoder still requires a separately approved
bounded GPU test before rollout. CPU/mock checks are not a hard latency guarantee.
Native decoding already running can finish after a client cancels;its result is
not allowed to restart the cancelled Jarvis conversation.
