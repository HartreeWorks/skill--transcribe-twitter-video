---
name: transcribe-twitter-video
description: This skill should be used when the user asks to "transcribe twitter video", "transcribe tweet", "transcribe this tweet", "get transcript from twitter", or provides a Twitter/X URL with intent to transcribe the video's audio. Downloads audio and invokes the transcribe-audio skill.
---

# Transcribe Twitter video

Download audio from Twitter/X video posts and transcribe using the transcribe-audio skill.

## Prerequisites

Ensure yt-dlp is installed:

```bash
brew install yt-dlp
```

The transcribe-audio skill must also be available (uses Parakeet MLX for transcription).

## Workflow

### Step 1: Validate the URL

Accept URLs in these formats:
- `https://x.com/username/status/1234567890`
- `https://twitter.com/username/status/1234567890`

Extract the tweet ID (the numeric portion after `/status/`).

### Step 2: Create output directories

```bash
mkdir -p ~/.claude/skills/transcribe-twitter-video/audio
mkdir -p ~/.claude/skills/transcribe-twitter-video/transcripts
```

### Step 3: Download audio

Use yt-dlp to extract audio only (more efficient than downloading full video):

```bash
DATE=$(date +%Y-%m-%d)
TWEET_ID="<extracted-tweet-id>"

yt-dlp -x --audio-format mp3 \
  -o "$HOME/.claude/skills/transcribe-twitter-video/audio/${DATE}-${TWEET_ID}.%(ext)s" \
  "<TWITTER_URL>"
```

The audio file will be saved as `YYYY-MM-DD-<tweet-id>.mp3`.

### Step 4: Invoke transcribe-audio skill

After downloading the audio, invoke the transcribe-audio skill:

```
transcribe ~/.claude/skills/transcribe-twitter-video/audio/<filename>.mp3
```

The transcribe-audio skill will:
1. Transcribe using Parakeet MLX (local, fast)
2. Perform speaker diarisation with FluidAudio
3. Generate a markdown transcript with speaker labels

### Step 5: Save transcript

Copy or move the generated transcript to the transcripts directory:

```bash
cp "<transcript-path>.md" ~/.claude/skills/transcribe-twitter-video/transcripts/
```

### Step 6: Report results

After transcription completes, report:
- Path to the transcript file
- Brief preview of the transcript content (first few lines)

## Output locations

- **Audio files**: `~/.claude/skills/transcribe-twitter-video/audio/`
- **Transcripts**: `~/.claude/skills/transcribe-twitter-video/transcripts/`

## Troubleshooting

If yt-dlp fails to download, try with browser cookies:

```bash
yt-dlp --cookies-from-browser chrome -x --audio-format mp3 \
  -o "$HOME/.claude/skills/transcribe-twitter-video/audio/${DATE}-${TWEET_ID}.%(ext)s" \
  "<TWITTER_URL>"
```

If the video has no audio track, yt-dlp will report an error. In this case, inform the user that the tweet video has no audio to transcribe.

## Update check

This skill is managed by [skills.sh](https://skills.sh). To check for updates, run `npx skills update`.
