---
name: Culturecards MCP
description: Install and use the Culturecards MCP server; understand its OAuth setup, tools, and edition content hierarchy including chapter, card, and slide types.
---

# Culturecards MCP and content model

Use this skill when setting up a Culturecards MCP client or working with Culturecards editions, chapters, cards, slides, or media through MCP.

## MCP server URL

Culturecards exposes a remote Streamable HTTP MCP server. Use the URL for the desired environment:

- Production: `https://mcp.culturecards.io/`
- Local development: `http://localhost:3003/`

The server uses Supabase Auth OAuth 2.1. The user authenticates and approves access through the matching Studio `/oauth/consent` flow; the server forwards the user's token to the API, which enforces permissions and edition access. The user needs an existing Studio profile and role. Never request or configure a user's token manually, and never substitute a privileged credential. Local use requires the local Supabase, API, Studio, and MCP services to be running.

## How the content hierarchy works

```text
Edition
└── Chapters (ordered)
    └── Cards (ordered within a chapter)
        └── Slides (one ordered sequence per card)
```

- **Edition** is the publication and access boundary. It contains edition metadata and the full content collection. An edition has a published version and an editable draft. Publish explicitly to make draft content public.
- **Chapter** groups cards within one edition and provides ordering/grouping for navigation. Chapters do not have an active/inactive flag.
- **Card** is the user-facing, shareable/QR-linked content unit within a chapter. It has its own metadata, active state, and one or more slides. Inactive cards can still appear in management tools; public visibility follows the API's active-content rules.
- **Slide** is a page/step of a card. A card can mix slide types; all its slides share a single order. The sort API expresses ordered positions starting at 1; stored `sort` values can have gaps, so preserve relative order rather than assuming contiguous stored values.
- **Media** is a reusable uploaded asset referenced by cards or slides. Upload media first and use the returned media ID in content fields. Subtitle SRT files for audio are stored separately, keyed to the audio media ID; they are not media records or slide fields.

The hierarchy is owned by its parent: chapters belong to editions, cards belong to chapters and editions, and slides belong to cards. Do not invent parent links or pass internal draft relationship IDs. Use IDs returned by MCP/API responses (public/published IDs for existing linked content; the entity ID returned for never-published content).

### Editorial workflow: turn source material into an edition

When a user supplies notes, documents, media, or a broad subject for an edition, do not copy it into records as one long description or create arbitrary divisions. First understand the intended audience, purpose, scope, key claims, sources, and required coverage. Ask focused questions when those are unclear or when source material conflicts. Preserve factual meaning and provenance; do not invent facts to fill gaps.

Plan from the whole to the parts:

1. **Find the narrative spine.** Summarize what the audience should understand by the end. Identify the main themes, chronology, locations, people, evidence, turning points, and present-day relevance. Choose a thematic, chronological, geographic, or mixed organizing principle that best serves the subject.
2. **Create chapters as meaningful arcs.** Each chapter should answer a distinct organizing question or cover a coherent period, theme, or place. Give it a clear title and brief description explaining its role in the overall journey. Arrange chapters so the sequence builds understanding; avoid chapters that are merely arbitrary length buckets or duplicate each other.
3. **Create cards as focused story units.** A card should center on one person, object, place, event, idea, or closely related set of evidence. It should make sense when opened on its own while clearly belonging to its chapter. Give it a concise identifying title, an optional clarifying subtitle, a short standalone synopsis, and an appropriate lead image/media asset when available. Keep the card description brief, plain text, and free of Markdown; it may be read aloud. Split a card when it tries to tell several independent stories; combine material when splitting would destroy necessary context.
4. **Write slides as a purposeful sequence.** Each slide is one step in the card's story, not a miniature unrelated card. A useful flexible progression is: hook or orient the reader; identify what they are seeing; explain relevant context; develop evidence, events, or human experience; show consequence or change; and close with significance, memory, or a question that leads onward. Use only the beats the story needs. Prefer one main idea per slide, concise reader-facing text, informative titles, and media that supports the point being made. Keep slide text brief, plain text, and free of Markdown or formatting markup because it may be sent to text-to-speech; write numbers, abbreviations, and punctuation so they sound natural when spoken. Define unfamiliar terms in context and distinguish evidence from interpretation.
5. **Choose and prepare images for phone screens.** Slides are designed to be viewed on iPhone 14 and newer, so prefer sharp, high-resolution portrait images close to a 9:17 aspect ratio (roughly 0.53:1), which fills a tall phone screen better than landscape or square images. Use at least 1080 × 1920 pixels for portrait images; prefer 1440 × 2560 pixels or higher when available, while keeping approximately the same aspect ratio. Use a different ratio only when the source image or story requires it; avoid stretching, excessive cropping, and images where important subjects or details will be lost in a phone-sized view. Choose clear, relevant images with sufficient resolution for modern high-density displays.
6. **Check the collection as a whole.** Verify that chapter order makes sense, every card has the right parent, stories are not duplicated or missing essential context, transitions are understandable, and media has usable alt text/credits where applicable. For image copyright information, provide links to the image sources; licensing statements or license labels are not relevant. Consider accessibility, audience suitability, and sensitive topics. Keep an introduction or conclusion as ordinary cards/slides if useful; the schema has no special intro/conclusion entity type.

Before creating a large set of records, present a proposed outline in this shape: edition purpose; ordered chapters with their role; cards under each chapter with title and synopsis; and each card's planned slide titles/beats, type, and needed media. Check that each supplied source item is assigned to the right card/slide or explicitly marked as unused, and call out missing evidence or editorial choices. Refine the outline with the user, then create in parent-first order (edition → chapters → cards → slides), using returned IDs and the MCP/API schemas. Do not make up media IDs: upload/select media and use the returned IDs. Ordinary edits still require the current `expected_version`; sorting requires the latest `expected_structural_version`.

Treat these as editorial guidance, not rigid quotas or a required slide template. The right number and structure depend on the amount and complexity of the source material. Do not create filler cards or slides merely to reach a target count. Do not force an even distribution. Let the material, narrative pacing, and reader navigation determine the size of each part. Split or combine content to improve coherence and usability, not to make every chapter or card the same length or count.

### Content versus delivery (MCP/API transport)

The hierarchy is editorial; MCP tools transport and mutate its records. An edition response contains its metadata and version tokens. `list_chapters(edition_id)` returns chapter records ordered by chapter `sort`. `list_cards(editionId)` takes an explicit edition ID and returns the edition's cards (including inactive cards); group them by `chapter_id` to recover the hierarchy, then sort cards by their order within each chapter. Card records expose synopsis/metadata and associated media; `get_card(id)` is the useful retrieval for the card's detailed ordered slide sequence. `get_slide(card_id, id)` also requires the parent card ID. Slide titles and text, media references, `type`, and `sort` together determine the particular slide's content and presentation. Stored sort values can have gaps; compare their ordering instead of assuming they are contiguous. Do not assume a card ID alone lets you locate a slide without reading that card's slides.

MCP tools call the generated Culturecards SDK, which calls the general Culturecards API with the authenticated user's bearer token. Do not use public-app `/user/*` endpoints as a shortcut: they have different visibility rules and omit inactive editorial content. The API authorizes each call, so an edition only appears when the caller can access it. To understand or edit content, fetch the parent edition and chapter/card relationships first, then inspect the relevant card and slides; do not infer story order from object creation times or UUIDs.

### Slide types

- **Default (`default`)**: image-led slide with text and optional image/media and voice. Text-to-speech can be regenerated from saved text. Saving a slide does not generate speech automatically.
- **Video (`video`)**: video-led slide with optional text, a video media reference, and an optional image/poster reference.
- **Audio (`audio`)**: audio-led slide referencing audio media, with an optional image/poster. Optional subtitles are an SRT file associated with that audio media ID, not a field on the slide.

These are distinct slide types, but sort them together in their one card-level order. TTS is for default slides; subtitle generation/upload is for audio media.

#### Voices for default slides

Default-slide voices are `ash` (Ben), `misty` (Laura), `rocco` (Mila Winter), and `jessie` (Joy). Use these keys—not the display names—in the slide's `voice` field or when requesting TTS regeneration. A null or unset voice uses `ash`; `default` is not a valid voice key. Voice selection is specific to default slides. Regenerating TTS uses the saved slide text and voice unless an optional voice override is supplied; saving a text or voice change does not itself regenerate speech. Check the live MCP/API schemas for the current accepted values.

### Choosing a slide type

- Prefer **default** slides for ordinary editorial content. Use them for explanations, narration, quotations, timelines, and image-led storytelling, including stories that discuss sound or video without supplying playable audio/video.
- Use **audio** only when the source includes explicit audio content that should be listened to, such as an interview, recording, or sound excerpt. Do not use audio just because a story mentions music, speech, or sound.
- Use **video** only when the source includes explicit video footage that should be watched. Do not use video for a still image, a written account of an event, or merely because a subject is visual.
- Do not make an edition or card all-video (or all-audio) by default. Choose the type slide by slide based on the actual source material and intended experience; when there is no supplied, relevant playable media, use a default slide.
- A media asset's file type alone is not enough reason to choose a media-led slide: confirm that the asset is relevant and intended as the slide's primary audio/video experience. Use default slides for the surrounding context and interpretation.

## Safe operating workflow

1. Discover the user's accessible editions with `list_editions`; inspect an edition using `get_edition`.
2. Read chapter/card/slide data before changing it. `list_cards` requires an explicit `editionId`; it does not use a selected/default edition. `get_slide` requires both card ID and slide ID. Before updating a slide, inspect its `type` and use the matching typed tool: `update_default_slide`, `update_video_slide`, or `update_audio_slide`. Do not use one slide-type update tool for another type; follow that tool's live schema for the fields it accepts.
3. For updates, use the latest `expected_version` from the entity response, including when calling a typed slide update tool. For `sort_edition`, use `expected_structural_version` from `get_edition` and send complete affected lists. When moving an item, include both source and destination lists. All slide types share the same ordered list.
4. Create parents before children: edition, chapter, card, then slide. Use returned IDs and required parent IDs/order values.
5. Review changes, then publish through the appropriate Studio/API workflow. Editing a draft does not itself publish it.
6. Re-read after a version conflict (HTTP 409) and reconcile against current state; never guess version tokens.

### Destructive operations and side effects

- `reset_edition` is destructive: it discards the entire unpublished draft (edits, additions, and deletions) and recreates it from published content, or initializes a missing draft. Published content/media stay unchanged. Confirm with the user first, then refetch entities and version tokens.
- Deleting a chapter also deletes its cards and slides; deleting a card deletes its slides. Confirm cascades before proceeding.
- Deleting a slide removes that slide. Uploading a file to an existing edition path may overwrite the existing file; prefer unique file names.
- `generate_subtitles` and `upload_subtitles` replace the SRT associated with that audio media.
- `generate_media_alt_text` only suggests alt text; it does not save it.
- `recreate_slide_tts` regenerates speech for a saved default slide; it does not save slide edits.

## Available MCP tools

The server exposes tools for media upload (`upload_media`, `upload_media_by_url`), listing/getting/updating editions, chapters, and cards, and slide updates split by type (`update_default_slide`, `update_video_slide`, `update_audio_slide`), plus atomic ordering (`sort_edition`), QR SVG retrieval, TTS regeneration, subtitle generation/upload, media alt-text suggestions, and deletion. Tool names and request schemas can evolve; discover the server's live tools and treat their descriptions and API validation as authoritative. All calls are made as the authenticated caller and remain subject to their permissions.

## Server configuration (operators)

The deployed server is operated by Culturecards; end users should connect to its public URL rather than deploy a private copy. A self-hosted instance requires server-side `CULTURECARDS_API_URL`, `SUPABASE_PUBLISHABLE_KEY`, `SUPABASE_URL`, and public `MCP_RESOURCE_URL` settings. Never expose server credentials to an MCP client. OAuth consent is handled by the matching Studio environment at `/oauth/consent`.
