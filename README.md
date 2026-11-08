# Blotato social poster

Two Make scenarios turn an Instagram Reel URL in Airtable into a rewritten script, then a HeyGen avatar video distributed through Blotato.

These are sanitized exports of an existing project, with a recovered Airtable structure. They require reconnection and the fixes below before use. Importing and live publishing have not been tested.

## How it works

1. **Reel to draft:** watches new Airtable records, extracts the Reel transcript with Apify, and uses OpenAI to write a Script, Caption, and Overlay back to that record. Missing or short transcripts are logged in ErrorLog.
2. **Avatar to social:** watches modified Airtable records, generates a HeyGen avatar video, uploads it to Blotato, and routes it to Facebook, Instagram, LinkedIn, YouTube, Pinterest, TikTok, X, Bluesky, and Threads.

The second trigger is named “Approved” in the original, but it has no approval filter. Checking Approved is not currently what controls publishing. Follow the setup guide before enabling either scenario.

## Architecture

**Blotato is the publishing layer of this project.** Airtable holds the content, Make runs the automation, Apify extracts the source transcript, OpenAI writes the draft, and HeyGen creates the avatar video. Make then sends that video and the post copy through Blotato to the configured social accounts.

```mermaid
flowchart TD
    A["Airtable Videos: new ReelURL"] --> B["Make scenario 1: watch CreatedTime"]
    B --> C["Apify: extract Reel transcript"]
    C --> D["OpenAI: rewrite and structure draft"]
    D --> E["Same Airtable record: Script, Caption, Overlay"]
    C -. "Missing or short transcript" .-> L["Airtable ErrorLog"]
    E --> F["Make scenario 2: watch ModifiedTime"]
    F --> G["HeyGen: generate avatar video from Script"]
    G --> H["Wait 300 seconds, then fetch video URL"]
    H --> I["Blotato: upload video"]
    I --> J["Make router: separate Blotato posting modules"]
    J --> K["Facebook, Instagram, LinkedIn, YouTube, Pinterest, TikTok, X, Bluesky, Threads"]
    J -. "Posting errors" .-> L
```

This diagram describes the recovered blueprints. The approval gate and publication tracking described below still need to be implemented.

### What each service does

| Component | Responsibility in this project |
| --- | --- |
| Airtable — Videos table | Stores the source URL, generated draft, intended approval checkbox, timestamps, and errors. It is the shared record between the two scenarios. |
| Make | Watches Airtable, calls the other services, maps their outputs, and routes publishing requests. Both exported JSON files run here. |
| Apify | Extracts the Instagram Reel transcript used by the first scenario. |
| OpenAI | Rewrites the transcript and produces the Script, Caption, and Overlay fields. |
| HeyGen | Turns Script into a portrait avatar video using the configured avatar and voice. Overlay is mapped to the video title. |
| Blotato | Receives the generated video through its media module, then receives platform-specific publishing requests containing the media, post copy, and destination account settings. |
| GitHub | Stores the blueprints, table template, and documentation. It does not run the automation. |

n8n was used to access authorized connections during recovery. It is not part of the runtime architecture of these Make blueprints.

### Scenario 1: source Reel to editable draft

Add an Instagram Reel URL to **ReelURL** in Videos. The first Make scenario watches **CreatedTime**, passes the URL to Apify, and retrieves the transcript. For a usable transcript, OpenAI rewrites the content and returns structured draft fields. Make writes **Script**, **Caption**, and **Overlay** back to the same Airtable record. Missing or short transcripts are recorded in **ErrorLog**.

The record is the handoff between scenarios; the first scenario does not directly call the second. The exported first scenario also contains a two-iteration repeater, so it can generate the draft twice. Review this before running it.

### Scenario 2: draft to video to Blotato publishing

The second Make scenario watches **ModifiedTime** on Videos. It sends **Script** to HeyGen with your avatar and voice IDs, waits 300 seconds, and fetches the generated video's URL. The **Blotato media module** uploads that video. A Make router then feeds separate **Blotato Post modules** for the nine destinations shown above, using the uploaded media and the mapped copy from **Caption** and **Overlay** as configured for each destination.

Your social accounts must be connected in Blotato, and each posting branch must use the corresponding account ID and any required page or board settings. Those private identifiers and connection references were removed from the GitHub exports; reconnect them in Make after importing. Keep only the destination branches you intend to use.

### Approval, state, and errors

The intended operating flow is **add a Reel → generate a draft → review and approve → generate a video → publish through Blotato**. The recovered publishing watcher does **not** enforce the Approved checkbox: any qualifying record modification can enter the second scenario, including draft updates from scenario 1.

Before enabling publishing, add an approval/valid-script filter and processing/publication state. Track success per destination so a retry does not resend posts that already succeeded. Error handlers write to ErrorLog, and because ModifiedTime watches all fields, those writes can trigger the watcher again without state gating. The original fixed wait also needs video-readiness handling before the Blotato upload.

The recovered table has no video-job or publication-status fields, and **BackgroundURL** is retained but unused by these routes. See [Setup and operation](docs/setup.md) for the required changes and a controlled testing sequence.

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
