---
name: akool
description: Creates AI videos, images, voiceovers, talking avatars and face swaps with the user's Akool account. Use when the user says "make a video from this image", "generate an image of", "read this script aloud", "make this avatar say", "swap my face into this photo" or "how many Akool credits do I have". Not for video translation, lip-syncing an existing video, editing an existing video, or anything unrelated to Akool.
---

# Akool

Akool tools submit generation tasks that finish asynchronously. The tool descriptions are the source of truth for
parameters, statuses, result fields and prices; these rules cover how to run a job end to end.

## Routing

| User goal | Create tool | Poll tool |
| --- | --- | --- |
| Check credit balance | `account_credit` | — |
| Image from text, or from an image plus text | `image_create` | `image_info` |
| Video from an image | `video_image2video_create` | `video_result` |
| Video from text | `video_text2video_create` | `video_result` |
| Speech from text | `audio_tts` | `voice_result` |
| Photo that speaks an audio track | `video_talking_photo_create` | `video_result` |
| Avatar that speaks text or audio | `video_talking_avatar_create` | `video_result` |
| Face swap into an image or video | `faceswap_plus` | `faceswap_result` |
| Person in an image copies a video's motion | `video_motion_swap_create` | `video_result` |

To make a talking photo say text, run `audio_tts` first and pass the finished audio URL as `audio_url`.

## Rules

- **Credits first.** Before a batch or a long or expensive video, call `account_credit`, tell the user roughly what
  the job will cost, and ask for one confirmation covering the whole batch. If credits are insufficient, stop and
  point the user to https://akool.com/pricing; do not retry.
- **Ask once.** Collect every missing required input in a single message instead of asking one question at a time.
- **Never invent IDs.** Take `voice_id`, `avatar_id` and `model_name` only from the list tools: `voice_list`,
  `avatar_list`, `account_models` and `video_motion_swap_models`.
- **Public URLs only.** Media inputs must be public URLs. If you can run shell commands and the user has a local
  file, get an upload with `file_upload` and use the returned URL. Never pass a local path to a tool.
- **Poll, then stop.** After submitting, keep the returned task id and poll the matching tool from the routing table
  at the interval that tool's description gives. If the task is not finished after about 10 minutes, give the user the task id and stop polling.
- **Retry once.** If a task fails, read its error, fix the cause if you can, and retry at most once.
- **Deliver.** Give the user the result URL and the task id. Video URLs expire after about 7 days, so suggest
  downloading anything worth keeping.
