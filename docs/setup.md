# Setup and operation

## 1. Prepare the services

You need Make connections for Airtable, Apify, OpenAI, HeyGen, and Blotato. Connect your intended social accounts in Blotato and obtain their account IDs. Blotato's official Make tutorial explains where to find its API key and account IDs; HeyGen API access is separate from its web-app subscription. See the [official tutorial](https://help.blotato.com/integrations-and-automation-templates/make.com/make-ai-social-media-system).

These are Make blueprints, not n8n workflows. n8n was used to access existing authorized connections during recovery; it is not required to run the exported project.

## 2. Prepare Airtable

The owner's new Videos table already exists in Google Business Profile Posts. For another base, recreate the following fields. ReelURL is the primary field.

| Field | Airtable type | Purpose |
| --- | --- | --- |
| ReelURL | Single line text | Source Instagram Reel URL |
| BackgroundURL | Single line text | Retained from the original structure; unused by these routes |
| Script | Long text | Spoken text for the avatar |
| Caption | Long text | Social post copy |
| Overlay | Long text | Short title; used as HeyGen video title |
| ErrorLog | Long text | Extraction and posting failures |
| Approved | Checkbox | Intended human approval; must be wired into filtering |
| CreatedTime | Created time | First scenario's trigger |
| ModifiedTime | Last modified time, all fields | Second scenario's original trigger |

The CSV contains only headers for editable fields; create the two computed timestamp fields in Airtable. The table is empty because the source records could not be recovered.

## 3. Import and reconnect

1. Create two Make scenarios and import the matching JSON blueprint into each. Keep scheduling disabled.
2. Select your connections in every module requiring one.
3. In every Airtable module, select the destination base and Videos table, then reselect fields. Do this for both triggers, draft updates, and every posting error handler. Refresh mapped output tokens after selecting the new table: the exported field names replace source-specific field IDs, and Make may require remapping.
4. Set CreatedTime as the first trigger's time field. Review the second trigger as described below.
5. Replace all avatar, voice, social account, Facebook page, and Pinterest board placeholders with your own values where applicable. Configure only the publishing branches you intend to use.
6. Review the OpenAI prompts. The original writing style targets private medical practices, uses UK English, and includes a follow CTA. Adapt this to your audience.

## 4. Fix the original behavior before enabling

These are findings from inspecting the exported configuration, not fixes already applied to the live scenarios.

- **Approval is not enforced.** The second Airtable watcher has an empty formula/view and no Approved condition. Add a filter requiring approval, a nonempty valid Script, and a record that has not already been claimed or published. Reject the `NO_USABLE_TRANSCRIPT` sentinel.
- **Add processing and publication state.** Introduce explicit state fields and claim a record before paid video generation. Track success per destination, and retry only failed destinations. Approval alone does not prevent duplicate posts.
- **Avoid modification loops.** ModifiedTime currently watches all fields. ErrorLog writes can trigger the same record again. Use a carefully selected trigger field/view together with state gating so processing writes cannot restart a job.
- **The first scenario repeats twice.** Its repeater is configured for two iterations; this repeats draft generation rather than implementing retry behavior. Remove it or set it to one iteration unless repeated generation is intentional.
- **Wait for video readiness.** The second scenario sleeps for 300 seconds and checks HeyGen once. Replace this with bounded polling that handles pending, failed, and completed states before upload.
- **Correct X and Bluesky text mapping.** The existing slice expression is malformed. Rebuild and test the caption truncation mapping in Make's editor against the destination's current requirements.
- **Review each platform's settings.** The exports include public publishing settings and AI/synthetic-media settings on some destinations. Review audience, media format, disclosure, and account selection before an intentional test.

The recovered schema does not include the additional state fields suggested above. Add them when implementing these safeguards.

## 5. Test deliberately

1. With the publishing scenario disabled, add one ReelURL and run the draft scenario once. Confirm extraction, the Script/Caption/Overlay outputs, and ErrorLog behavior. The source uses the Apify Instagram transcript extractor actor `3C7L8IMQOkq3isV2Y` with synchronous execution and an additional 30-second wait; verify actor availability and input compatibility in your account.
2. Review the draft manually. Confirm an unapproved record and an invalid script cannot enter video generation after adding your filters.
3. Test HeyGen generation and readiness handling. The original request uses portrait 720 × 1280 video, captions enabled, and voice speed 1.1.
4. Enable only one intended publishing branch for a controlled test. Verify its destination and result, then confirm rerunning the same record does not publish again.
5. Test a failed destination and ensure logging does not restart the workflow or resend successful posts. Enable other branches and scheduling only after these checks pass.

## Recovery and validation limits

Both source scenarios were inactive at export. The package retains their connected module structure, routing, prompts, and error handlers. It is an archival starting point requiring configuration; it has not been round-trip imported or exercised with paid APIs or live social posting. JSON syntax and static references are checked locally. No publishing run was performed during recovery.
