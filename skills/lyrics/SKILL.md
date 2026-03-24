---
name: lyrics
description: Look up song lyrics and explain their meaning. Use when the user asks about song lyrics, what a song is about, wants to find lyrics for a specific song, or mentions a song title and asks for its content/meaning. Triggers on "lyrics", "what is this song about", "nội dung bài", "bài này nói về gì".
---

# Lyrics Lookup

## Process

1. **Search** — WebSearch for `{artist} "{song title}" lyrics site:genius.com`
2. **Fetch** — Try sources in order until lyrics found:
   - Genius URL from search results (best source — most complete)
   - `lyricsondemand.com/{artist}/{song_title}`
   - `azlyrics.com/lyrics/{artist}/{songtitle}.html`
   - `lyricstranslate.com` (good for non-English songs)
   - Vietnamese songs: `zingmp3.vn`, `nhaccuatui.com`, `loibaihat.biz`
   - Niche/indie: Bandcamp pages sometimes have lyrics in description
3. **Present** — Show a summary of the song's themes and meaning. Reference specific lines when possible.
4. **Language** — If the song is in a foreign language (Japanese, Korean, Vietnamese, etc.), look for English translations too.

## WebFetch prompt

Use: "Extract the complete song lyrics text only. Remove ads, navigation, and other non-lyrics content. Return just the lyrics."

## Fallback — Playwright

If WebFetch fails or returns a copyright block (model refuses to return lyrics), use Playwright as fallback:

1. Follow playwright-hygiene rules (`mkdir -p /tmp/playwright-session`, etc.)
2. Navigate to the lyrics page (prefer `nhaccuatui.com` for Vietnamese songs — lyrics are in the accessibility snapshot)
3. Use `browser_snapshot` — lyrics appear as `text:` nodes in the snapshot, no need for screenshots
4. Extract lyrics from the snapshot and present to user
5. Close browser and clean up when done

## Notes

- Genius is the primary source — it has the widest coverage including indie artists
- WebFetch may be blocked by copyright filters — try multiple sources
- For Vietnamese songs, Vietnamese lyrics sites often have better coverage than international ones
