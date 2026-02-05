# AI Construction Timelapse Generator (Before → Middle → After)
**This template was created by Alex Safari.**  
Subscribe to my YouTube channel for more AI automation tutorials:  
👉 https://www.youtube.com/@alexsafari1  

Set up video: https://youtu.be/lYkVvQAg5l4

## Overview
This workflow generates a realistic **AI timelapse video** from two images: an **Initial (Before)** frame and a **Final (After)** frame.

Instead of forcing the video model to jump straight from “before → after” (which causes morphing/flicker), the workflow first generates a **Middle Bridge Frame** (50–60% completion). Then it writes **three cohesive video prompts**, generates **three 8-second clips**, creates a matching music bed, and stitches everything into a final 24-second timelapse.


## The Human Process (what a person would normally do)
If you were doing this manually, you’d:
1. Collect a “before” image and an “after” image.
2. Imagine a believable halfway stage (exposed framing, tools out, unfinished edges).
3. Write a mini director plan for the progression (Phase 1, Phase 2, final reveal).
4. Generate clips, pick music, and edit them together.
5. Save the output links and mark the job as complete.

This workflow automates all of it.

---

## Workflow Breakdown (Node-by-Node)

### 1) Trigger
- **Manual Trigger** starts the workflow (you also have an optional **Form Trigger**, currently disabled).

### 2) Job Queue (Airtable)
- **Search records** pulls rows where `{Status}="Pending"` so this can run like a batch processor.

**Required Airtable fields:**
- `ID`
- `Status` (Pending / Done)
- `Initial Image` (attachment URL)
- `Final Image` (attachment URL)
- `Middle Image` (attachment URL)
- `Middle Image Prompt` (text)
- `First Video Prompt`, `Second Video Prompt`, `Third Video Prompt` (text)
- `Video Clip 1`, `Video Clip 2`, `Video Clip 3` (urls)
- `Music URL` (url)
- `Final Video` (url)

Get Airtable base here: https://airtable.com/appP9Tan7f1mONBsR/shruypLYIy5GxVVMH
---

### 3) Generate “Middle Image Prompt” (Bridge Frame)
- **Generate middle image prompt (Gemini 2.5 Flash)** analyzes the Initial + Final images.
- Outputs a single prompt describing the project at **50–60% completion**.

**Key constraints enforced by the prompt:**
- Locked-off static camera, identical angle to reference frames (no perspective drift)
- A sharp “work-in-progress” edge (finished material meets raw surface)
- Skeletal exposure (rebar/studs/joists/internal elements visible)
- Active staging (tools/materials/furniture in transit positions)
- Lighting buffer (time-of-day halfway between start and end)

---

### 4) Create the Middle Image (Image-to-Image)
- **Call ‘Fal.ai nanobanana2 image to image edit’** (subworkflow)
- Takes:
  - `prompt` (from Gemini)
  - `imageURLs` ([Initial, Final])
  - `aspect_ratio` (9:16)

Outputs:
- `Middle Image URL`

---

### 5) Save Middle Image + Prompt Back to Airtable
- **Update Middle Image**
- Stores:
  - `Middle Image` (attachment)
  - `Middle Image Prompt` (text)
  - Matches by `ID`

---

### 6) Generate 3 Video Prompts (Director-Style)
- **Generate video prompts (Gemini 2.5 Flash)** analyzes:
  - Initial frame
  - Middle frame
  - Final frame

And generates 3 prompts that follow a consistent cinematic style:

**Clip 1 (Start → Mid)**
- Construction/base layering
- High-speed motion blur workers
- Slow cinematic camera movement

**Clip 2 (Mid → End)**
- Finishing materials / refinement
- High-speed motion blur workers
- Slow cinematic camera movement

**Clip 3 (Final Reveal)**
- Pristine, no workers
- Slow cinematic push-in
- Calm, clean atmosphere

Each prompt includes realistic **SFX cues**.

---

### 7) Convert Prompts Into Structured Fields
- **Format Video Prompts (Agent)** uses:
  - **OpenRouter Chat Model**
  - **Structured Output Parser**

Returns:
- `clip1_Prompt`
- `clip2_Prompt`
- `clip3_Prompt`

---

### 8) Generate Clip 1 (Initial + Middle)
- **Call ‘Kie.ai VEO3.1 fast image to video subworkflow’**
- Inputs:
  - `video_prompt` = clip1
  - `imageUrls` = [Initial, Middle]
  - `generationType` = FIRST_AND_LAST_FRAMES_2_VIDEO
  - `aspect_ratio` = 9:16

Output:
- `Video Clip 1 URL`

---

### 9) Generate Clip 2 (Middle + Final)
- **Call ‘Kie.ai VEO3.1 fast image to video subworkflow’1**
- Inputs:
  - `video_prompt` = clip2
  - `imageUrls` = [Middle, Final]
  - `generationType` = FIRST_AND_LAST_FRAMES_2_VIDEO
  - `aspect_ratio` = 9:16

Output:
- `Video Clip 2 URL`

---

### 10) Generate Clip 3 (Final Reveal)
- **Call ‘Kie.ai VEO3.1 fast image to video subworkflow’2**
- Inputs:
  - `video_prompt` = clip3
  - `imageUrls` = [Final]
  - `generationType` = FIRST_AND_LAST_FRAMES_2_VIDEO
  - `aspect_ratio` = 9:16

Output:
- `Video Clip 3 URL`

---

### 11) Generate Music Prompt + Music
- **Music Prompt Generator (Agent)** creates a Suno-ready prompt based on:
  - “This is a 24 seconds video having three 8 seconds clips stitched together…”
  - The 3 video prompts

- **Generate Music (Kie.ai Suno text to music subworkflow)** generates the track.

Output:
- `Music URL`

---

### 12) Save All URLs + Prompts to Airtable
- **Update All URLs to Airtable**
- Stores:
  - Video Clip 1 / 2 / 3 URLs
  - Music URL
  - First / Second / Third Video Prompt text

---

### 13) Prepare Stitching Payload
- **Prepare Stitching Payload (Code Node)**
- Builds:
  - `video_urls` array [clip1, clip2, clip3]
  - `music_url`
  - packaged payload for your stitching subworkflow

---

### 14) Stitch Final Video
- **Stitch Final Video (Shotstack edit subworkflow)**
- Combines:
  - 3 clips + music
- Output:
  - `Final Video URL`

---

### 15) Save Final Video + Mark Done
- **Save Final Video**
- Updates Airtable:
  - `Final Video` URL
  - `Status` = Done

---

## ⚙️ Tech Stack
- **n8n** – orchestration & logic
- **Airtable** – job queue + logging
- **Gemini 2.5 Flash** – image understanding + prompt generation
- **Fal.ai (Nanobanana2)** – middle bridge image generation
- **Kie.ai (Veo 3.1)** – image-to-video for 3 clips
- **Kie.ai (Suno)** – background music generation
- **Shotstack** – stitching clips + music into a final output

---

📧 **Email me directly**: contact@loopsera.com  
For quick questions, customisation requests, or workflow troubleshooting.

🌐 **Visit my website**: https://loopsera.com  
Explore more automation templates, services, and case studies.

📞 **Book a Discovery Call**: https://cal.com/loopsera/ai-opportunity-audit 
For businesses that need a custom AI agent built around their content or operations.

🎓 **1-on-1 Coaching Session**: https://cal.com/loopsera/n8n-ai-agent-coaching-session  
Personalised coaching to help you build, troubleshoot, or optimise n8n AI workflows. Perfect for both beginners and advanced users. Includes live guidance and actionable tips.
