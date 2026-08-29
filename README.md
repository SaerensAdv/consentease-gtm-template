# ConsentEase CMP — Google Tag Manager Community Template

Official Google Tag Manager tag template for [ConsentEase](https://consentease.io),
the GDPR/CCPA consent management platform. The template loads the ConsentEase
cookie banner and manages [Google Consent Mode v2](https://developers.google.com/tag-platform/security/guides/consent)
for all Google and consent-aware third-party tags — codeless.

> **Status:** public repository prepared for submission to the Community Template
> Gallery. Template Editor validation and Google approval are still pending; see
> the checklist at the bottom.

## What the template does

Fired on the **Consent Initialization - All Pages** trigger, the template:

1. Validates the configured **ConsentEase Site ID** (`publicId`).
2. Sends the Consent Mode v2 **defaults exactly once**: all signals `denied`,
   `security_storage` `granted`, with a configurable `wait_for_update`
   (default 500 ms).
3. Optionally sets `ads_data_redaction` and `url_passthrough` (both default on,
   matching the manual ConsentEase embed snippet).
4. Registers the **ConsentEase banner bridge (protocol v1)** on
   `window.__consentEaseGtmBridge` — *before* the banner loads.
5. Injects the real banner script
   `https://consentease.io/api/consent/{publicId}/script.js`.
6. Applies consent updates the banner publishes through the bridge —
   deduplicated, so each semantically unique consent status is applied
   **exactly once**.

### Bridge protocol v1

The banner detects the bridge (same protocol version, same Site ID, `publish`
function) and then leaves all Google Consent Mode communication to the
template: no duplicate defaults, no duplicate updates. Payload shape:

```json
{
  "protocolVersion": 1,
  "publicId": "<Site ID>",
  "eventId": "ce-<Site ID>-<n>",
  "source": "consentease-banner",
  "consent": { "necessary": true, "functional": false, "analytics": true, "marketing": false }
}
```

Consent category mapping (identical on both sides of the bridge):

| ConsentEase category | Google Consent Mode signals |
| --- | --- |
| functional | `functionality_storage`, `personalization_storage` |
| analytics | `analytics_storage` |
| marketing | `ad_storage`, `ad_user_data`, `ad_personalization` |
| — | `security_storage` is always `granted` |

An incompatible bridge (different protocol version or Site ID) is reported
clearly in the console; the banner then runs standalone and **never** grants
consent automatically. Consent storage (`ce_consent_{publicId}`, a
`{ consent, expires }` wrapper) is owned entirely by the banner — the template
never reads cookies or localStorage.

## Installation (Template Editor, pre-Gallery)

1. In Google Tag Manager open **Templates → Tag Templates → New**.
2. Menu (⋮) → **Import**, select `template.tpl`, then **Save**.
3. Run the built-in tests: **Tests** tab → **Run all tests** (all must pass).
4. Create a tag from the template, enter your **ConsentEase Site ID** (found in
   your ConsentEase dashboard under *Websites → Installation*).
5. Set the firing trigger to **Consent Initialization - All Pages**.
6. Remove any manual ConsentEase snippet from the page — the template and the
   manual snippet are alternatives, never both.

## Parameters

| Parameter | Default | Description |
| --- | --- | --- |
| `publicId` | — | ConsentEase Site ID (letters, digits, `-`, `_`; 6–64 chars). |
| `ads_data_redaction` | `true` | Redact ad identifiers while `ad_storage` is denied. |
| `url_passthrough` | `true` | Pass ad click info through URL parameters while consent is denied. |
| `wait_for_update` | `500` | Milliseconds Google tags wait for a consent update. |

## Permissions used

- **Accesses consent state** (write-only) for the seven Consent Mode v2 signals.
- **Writes data layer** for `ads_data_redaction` and `url_passthrough`.
- **Accesses global variables**: `__consentEaseGtmBridge` (write),
  `__consentEaseGtmBridgeInfo` (read/write, duplicate-load guard).
- **Injects scripts** — restricted to `https://consentease.io/api/consent/*`.
- **Logs to console** — debug environments only.

## developer_id — intentionally disabled

This template **does not set** `gtagSet('developer_id.…')`. Google has not
assigned a developer ID to ConsentEase yet. It must only be added after Google
officially assigns one — never invent it.

## Gallery submission checklist

- [x] Push the contents of this directory to the dedicated **public** GitHub
      repository (repository root = these files).
- [x] Confirm `homepage` and `documentation` URLs are live.
- [x] Fill in `versions` in `metadata.yaml` with the real release commit SHA.
- [ ] Run all Template Editor tests against the imported `template.tpl`.
- [ ] Submit the repository via the Community Template Gallery.
- [ ] After approval: record `galleryOwner`/`galleryRepository` in the
      ConsentEase server configuration (see `docs/gtm/api-contract.md`).
- [ ] Request a developer ID from Google; only then enable it in the template.

## License

[Apache 2.0](./LICENSE)
