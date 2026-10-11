---
name: content-creation
description: Create content for artists and record labels — short-form videos, images, captions, lipsync clips, and more. Use when the user asks to create content, make a video, generate an image, produce a TikTok or Reel, create a promotional clip, add captions, or create visual content for an artist. Also use when iterating on content quality — regenerating images, trying different audio, adjusting text, or upscaling for higher quality.
---

# Content primitives

Read `references/content-api.md` for the supported request contracts, hosted presets, polling and costs. Direct video uses H3 first/last-frame inputs; hosted lipsync is an existing `/content/create` preset whose audio-conditioning repair requires the coordinated API/worker release. There are no six-mode CLI/API examples to copy. Verify artifacts before delivery; inspect motion and actual audio, and report unverified lip alignment.
