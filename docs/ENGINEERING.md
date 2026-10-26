# Engineering decisions

[Back to the overview](../README.md)

Far0 combines a native local library with a separate capture process and optional remote article processing. These notes describe the design at a high level; the implementation remains private.

## Local data ownership

The main app owns the SwiftData library, with CloudKit explicitly disabled. Bookmarks, notes and offline article copies remain on the device. SwiftUI provides navigation and layouts for iPhone and iPad.

If the persistent library cannot open, the app offers a retry without deleting or silently replacing the data store. Subscription expiration preserves saved content and pending captures.

## Capture across app processes

The Share Extension writes a small payload into a shared App Group queue instead of opening the app's model container. The payload includes a unique capture ID, URL, destination, note and timestamp.

Atomic writes protect the handoff. The app validates URLs, recognizes replayed capture IDs and saves imported data before removing the queued payload. Failed imports can be retried. An explicitly selected inbox destination is preserved during later organization.

## Metadata and offline reading

Asynchronous enrichment retrieves available titles, descriptions and images, with dedicated handling for some sources. Metadata runs before optional summary work. Personal titles and manual organization are preserved.

Network failure does not make a stored bookmark disappear. Failed previews can be retried. Offline reading stores extracted article text, retaining an existing local copy if refreshing it cannot be saved.

## Optional remote processing

With separate consent, the app uploads downloaded page HTML, its URL and title. A Supabase Edge Function uses Readability to extract article text and Groq to generate a structured result. The client validates the returned summary before saving it locally. Provider and service-role credentials stay on the server.

The design bounds request size, extracted input and generated output, reuses saved summaries and limits beta provider attempts. Automated tests simulate AI responses rather than consume live inference. Personal notes are outside the summary payload.

Production identity, server-side purchase authorization, per-user quotas and a global spending budget remain release work. Client checks and consent alone do not establish subscriber authorization at the backend.

## Access and privacy boundaries

StoreKit and RevenueCat provide the subscription integration. The extension checks current verified Apple transactions when evaluating capture access. Test-store behaviour does not validate real Apple subscription lifecycles. Production purchases remain blocked.

Usage statistics are optional and disabled in Debug. Explicit events exclude library content. Reminders use local notifications rather than a remote push service.

## Validation approach

The private project has regression checks covering persistence and migrations, capture import and retry, metadata, search, offline reading, deletion, subscription states, trial eligibility, consent, analytics and Rediscover. Backend tests use simulated responses. The macOS check workflow includes an unsigned Release build for an iOS simulator.

This showcase update checks its own file inventory, local documentation links, image references and public contents. It does not claim a fresh full application regression run. Simulator screenshots demonstrate the displayed UI, not physical-device behaviour, production purchases, accessibility certification or readiness of the deployed backend.
