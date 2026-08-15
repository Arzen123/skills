---
name: dnp-clinical-sites
description: Find candidate FNP/primary-care clinical placement sites in a California region and output them as a KML file for import into Google My Maps. User-invoked only.
disable-model-invocation: true
---

Find candidate clinical placement sites for a DNP-FNP student rotation in a
given California region, and write them out as a KML file the user can
import into Google My Maps as a new layer.

## 0. Announce version

Read the `VERSION` file in this skill's base directory and print its
contents as the first line of output, e.g. `dnp-clinical-sites v<contents
of VERSION>`.

## 1. Gather run inputs

Ask for whatever wasn't already given in the invocation:

- **Region** (required): a specific California city, county, or metro area.
  California as a whole is too broad for one run — if the user gives
  something that broad, ask them to narrow it.
- **Target count**: how many candidate sites to find. Default to 10-15 if
  not specified.
- **Known sites** (optional): a pasted list of site names already on their
  map. If given, skip obvious name matches during search so the output
  doesn't duplicate what they already have.

## 2. Search for candidates

Focus on **FNP / primary-care** appropriate site types only — this is not a
multi-specialty rotation search:

- Federally Qualified Health Centers (FQHCs) and community health centers.
  Check HRSA's Find a Health Center directory
  (https://findahealthcenter.hrsa.gov) for the region — it's public, free,
  and reliably has name + address for every listed center.
- Rural health clinics.
- Primary care / family medicine practices.
- Urgent care clinics that employ NP preceptors.

Skip hospital systems and specialty-only sites unless they have a clearly
primary-care-oriented outpatient arm. Use general web search for the
non-HRSA sources, e.g. "`<region>` federally qualified health center",
"`<region>` family medicine clinic", "`<region>` rural health clinic nurse
practitioner".

These are unverified leads, not confirmed preceptorships — the skill is
surfacing plausible candidates for the user to vet themselves, not
confirming any affiliation agreement exists.

## 3. Capture per-site details

For each candidate, record:

- Site name
- Full street address (required — this is what gets geocoded on import)
- Phone number, if found
- Website, if found
- One-line note on why it's a plausible FNP primary-care placement
- Source URL used to find it

## 4. Build the KML file

Follow `references/template.kml` exactly for structure. Key rules:

- One `<Document>` containing a single `<Folder>` named
  `<Region> - <today's date>` (e.g. `Sacramento County - 2026-08-14`), so it
  imports as one identifiable layer in Google My Maps.
- One `<Placemark>` per site, with `<name>` and an `<address>` element.
- **Never fabricate `<coordinates>` or lat/long.** Only general web search
  is available here — there's no geocoding step. Google My Maps geocodes
  plain `<address>` text automatically on KML import, so leave coordinates
  out entirely and let the address element do the work.
- Put phone, website, the placement note, and the source URL in
  `<description>`, one per line.

## 5. Write the output and hand off

Ask the user where to save the file (default to Desktop, or a dedicated
output folder if they have one). Name the file after the region and date,
e.g. `sacramento-county-2026-08-14.kml`.

This user's Desktop is OneDrive Known Folder-redirected: the real path
Explorer shows as "Desktop" is `C:\Users\singh\OneDrive\Desktop`, not
`C:\Users\singh\Desktop`. When defaulting to Desktop, write there — or
confirm first with `[Environment]::GetFolderPath("Desktop")` in PowerShell,
which resolves the redirect correctly.

Tell the user how to use it:

1. Open Google My Maps → the target map → **Import**.
2. Select the generated KML file. Google will geocode each `<address>`
   automatically and add one placemark per site.
3. Check the new layer against existing layers for obvious duplicates
   before keeping it — this skill doesn't dedupe against sites already on
   the map since that map isn't something it can read.
