# MiniMax H3 Speed LoRAs: ComfyUI workflows

These are the ComfyUI workflows from the Beyond the Frame video **"MiniMax H3: How to Be Fast and Still Hit (Speed LoRAs, Quality, Voice)"**.

Turbo LoRAs make MiniMax H3 fast but sloppy. The video covers three fixes:

| Chapter | Problem | Fix | Workflow |
|---|---|---|---|
| I · Speed | The Turbo LoRA won't load, or loads badly | Use the pruned ComfyUI file: strength 1.0, 8 steps, euler / beta | any H3 workflow |
| II · Quality | Melting hands and motion that falls apart | 6 clean steps, then 3 TaoMate steps at 0.7 with no new noise | `H3_TaoMate_refiner_6plus3.json` |
| III · Voice | Robotic, metallic voice | 10 steps, shift video 6 / audio 4, plus a clean draft audio as the reference | `H3_voice_fix_draft_audio_ref.json` |

> These tips come from community threads and my own tests. They're suggestions, not a guaranteed fix. Every setup is different, so test them on your own rig and keep what works for you.

---

## Downloads

| What | Where | File |
|---|---|---|
| Turbo LoRA, pruned for ComfyUI (drbaph) | [drbaph/MiniMax-H3-Turbo-Lora-ComfyUI](https://huggingface.co/drbaph/MiniMax-H3-Turbo-Lora-ComfyUI) | `minimax_h3_turbo_v4_step600_ema_pruned_comfyui.safetensors` |
| TaoMate 3-step, ComfyUI version (drbaph) | [experimental folder](https://huggingface.co/drbaph/MiniMax-H3-Turbo-Lora-ComfyUI/tree/main/experimental) | `minimax_h3_taomate_fl2va_3step_ema_comfyui.safetensors` |
| Turbo Sampler + H3 Turbo nodes (larryvrh) | [Larryvrh/ComfyUI-MiniMax-H3-Turbo](https://github.com/Larryvrh/ComfyUI-MiniMax-H3-Turbo) | install with ComfyUI-Manager |
| Original Turbo LoRA (larryvrh) | [larryvrh/MiniMax-H3-Turbo-Lora](https://huggingface.co/larryvrh/MiniMax-H3-Turbo-Lora) | not the ComfyUI version |
| Original TaoMate (Alibaba TaoLive AIGC team) | [TaoLiveAIGC/TaoMate-H3](https://huggingface.co/TaoLiveAIGC/TaoMate-H3) | for their own Python runtime, not ComfyUI |

Put the LoRA files in `ComfyUI/models/loras`.

You also need the MiniMax H3 base model, text encoder and the video and audio VAEs. The workflows use my local file names, so after loading one, **re-select your own files** in the loader nodes (UNET, CLIP, VAE, LoRA).

---

## Chapter I · Turbo LoRA, set up right

Use the pruned file. The unpruned LoRA carries adapters that have nothing to attach to in the ComfyUI model, so it can fail to load or load only partly. The pruned file drops them, so every adapter attaches cleanly.

- `minimax_h3_turbo_v4_step600_ema_pruned_comfyui.safetensors`
- strength **1.0**
- **8 steps**
- sampler **euler**, scheduler **beta**

## Chapter II · `H3_TaoMate_refiner_6plus3.json`

Turbo LoRAs decide the motion in very few steps, which is where bodies melt and the motion falls apart. This workflow lets the plain model decide the motion first, then uses TaoMate only to finish the detail.

1. One **BasicScheduler** makes a single 9-step schedule (simple).
2. **SplitSigmas** cuts it after step 6.
3. **Pass 1** runs steps 1–6 on the plain model, with no LoRA. This is where motion, bodies, physics and sound get decided.
4. **Pass 2** runs steps 7–9 with **TaoMate at 0.7** through **DisableNoise**. It adds no new noise, so it carries on from where pass 1 stopped instead of starting over.

Keep the seed fixed and compare against an 8-step Turbo render of the same prompt. 608×352 is just the fast test size, so raise it for finals.

## Chapter III · `H3_voice_fix_draft_audio_ref.json`

Few-step renders make the voice sound robotic. There are two fixes, and this workflow uses both.

**Fix 1: steps and shift.** Run 10 steps with the larryvrh **Turbo Sampler**. In the **ModelSamplingMiniMaxH3** node, right after the Turbo LoRA, set **shift video 6 / shift audio 4**. The stock values are 12 / 3.

**Fix 2: a clean voice to follow.**
- **Stage 1:** the same prompt at **32×32**, **20 steps**, no LoRA. The picture is useless; keep **only the audio**.
- **Stage 2:** the Turbo render through **MiniMaxH3ReferenceToVideo**, with the draft audio plugged into `ref_audio_0`. In your prompt, tell it to follow `<Audio 1>`.

You get a fast picture with slow, clean sound. Both stages share one seed.

---

## Credits

- **drbaph:** pruned ComfyUI conversions of the Turbo LoRA and TaoMate
- **larryvrh:** the original Turbo LoRA and the ComfyUI Turbo nodes
- **Alibaba TaoLive AIGC team:** TaoMate-H3
- **MiniMax:** the H3 model

The LoRAs keep their own licences; check each model page before using them commercially.

Made by **Beyond the Frame**. If this saved you some render time, subscribe for more AI video breakdowns.
