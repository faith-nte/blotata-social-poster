# Blotato social poster

Two Make scenarios turn an Instagram Reel URL in Airtable into a rewritten script, then a HeyGen avatar video distributed through Blotato.

These are sanitized exports of an existing project, with a recovered Airtable structure. They require reconnection and the fixes below before use. Importing and live publishing have not been tested.

## How it works

1. **Reel to draft:** watches new Airtable records, extracts the Reel transcript with Apify, and uses OpenAI to write a Script, Caption, and Overlay back to that record. Missing or short transcripts are logged in ErrorLog.
2. **Avatar to social:** watches modified Airtable records, generates a HeyGen avatar video, uploads it to Blotato, and routes it to Facebook, Instagram, LinkedIn, YouTube, Pinterest, TikTok, X, Bluesky, and Threads.

The second trigger is named “Approved” in the original, but it has no approval filter. Checking Approved is not currently what controls publishing. Follow the setup guide before enabling either scenario.

## Files

- `blueprints/01-reel-to-draft.json` — Make scenario 8198338.
- `blueprints/02-avatar-to-social.json` — Make scenario 8199854.
- `airtable/schema.json` — recovered field specification; a reference, not a complete Airtable API creation payload.
- `airtable/videos-template.csv` — empty CSV header for editable fields.
- [Setup and operation](docs/setup.md) — connections, field mapping, required fixes, and testing.

The Videos table was recreated in the owner's Google Business Profile Posts base on 8 November 2026. The original records were unavailable; no historical rows are included. The live scenarios were not repointed or enabled.

## Related official tutorial

Blotato's [Make AI Social Media System tutorial](https://help.blotato.com/integrations-and-automation-templates/make.com/make-ai-social-media-system) explains importing Make blueprints, connecting Blotato and HeyGen, and configuring accounts. Its example uses Google Sheets and articles, so it is a related reference rather than the exact Airtable/Reels project recovered here. See its [video walkthrough](https://www.youtube.com/watch?v=4FAHV19KwKQ) and the [Blotato API documentation](https://help.blotato.com/api/start).

## Export handling

Credentials, Make connection references, cached sample data, original Airtable IDs, and social account/avatar/voice identifiers were removed or replaced with placeholders. Disconnected designer-only modules were omitted. Connected routes and error handlers were preserved, including the limitations documented in the setup guide. Store credentials in service connections, never in this repository.
