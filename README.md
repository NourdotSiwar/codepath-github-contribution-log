# Contribution 552: When matching existing audiobooks, search results should show the narrator

**Contribution Number:** 552

**Student:** Nour Siwar

**Issue:** [GitHub issue link](https://github.com/Listenarrs/Listenarr/issues/552)

**Status:** Phase IV - Complete

**Fork Link:** [Listenarr](https://github.com/NourdotSiwar/Listenarr)

---

## Why I Chose This Issue

I like to read Fanfiction as a hobby and I found the project to be related to AudioBooks which I found interesting. 

This issue interests me for several reasons:

1. It is a simple feature which will help me ease into contributing to open source.
2. It is easy to understand and it matches my skills in terms of C#
3. I can ask Claude about Vue and TS to debug any problems while working on this feature since I do not know these languages yet.

This is my first time contributing to open source. So I am hoping to learn to understand new codebases with the help of Claude as well as learn more about how different languages work with each other such as backend like C3 with frontend like TS. 

---

## Understanding the Issue

### Problem Description

When matching or importing an audiobook that already exists in the library, Listenarr
shows a list of candidate metadata results to choose from — but those results don't
display the narrator. Many audiobooks have multiple recordings of the same title read
by different narrators, so without the narrator shown, there's no reliable way to tell
the candidates apart and pick the correct edition. The issue's screenshot contrasts
this with Audiobookshelf's match screen, which surfaces the narrator and publication
year for exactly this reason.

### Expected Behavior

When matching/importing an existing audiobook, each search result should display the
narrator (ideally alongside the publication year), so users can distinguish between
multiple recordings of the same book and select the right match.

### Current Behavior

The candidate match results show title, author, and cover, but omit the narrator,
making it ambiguous which recording is which when several share a title.

### Affected Components

This is primarily a **frontend** change in the Vue + TypeScript code under `fe/`,
not the C# backend. Key findings from exploring the codebase:

- The data is already available. The `SearchResult` type (`fe/src/types.ts`, backed by
  `listenarr.api/Models/SearchResult.cs`) already includes a narrator field, and the
  metadata responses carry narrator info.
- The "Add New" search flow (`fe/src/views/AddNewView.vue`) **already renders** the
  narrator on its result cards — so the existing pattern to copy is already in the repo.
- The gap is in the *existing-audiobook* match/identify flow, reached via the Edit
  action on a library item, most likely `fe/src/components/EditAudiobookModal.vue`
  (to be confirmed during reproduction). That component's results list is the one
  missing the narrator line.
- A backend touch in `listenarr.api/Services/SearchService.cs` may only be needed if
  the match search uses a lighter endpoint that doesn't populate narrator — to be
  verified by inspecting a result object during reproduction.

Languages/tools involved: Vue 3, TypeScript (primary); possibly a small amount of
C#/.NET if the backend enrichment turns out to be incomplete.

---

## Reproduction Process

### Environment Setup

Set up local dev environment on Windows with WSL2. Frontend runs via Node.js 24 (npm run dev). Backend runs separately. Challenges faced: 
  
- global.json requires .NET 10 SDK — resolved by running backend through existing Windows install
- WSL git ownership error — resolved with git config --global --add safe.directory
- Library Import scan requires real audio files — empty placeholder files are skipped by the backend scanner

### Steps to Reproduce

1. Open fe/src/components/domain/audiobook/LibraryImportSearchModal.vue lines 63–75
2. Observe the result card template: renders title, authors[0].name, series, and asin — narrator field is absent
3. Compare with fe/src/views/content/AddNewView.vue line 654: identical SearchResult data shape, but renders Narrated by {{ book.searchResult.narrator }}
4. Observed result: Narrator data exists in the SearchResult type but is never rendered in the match modal, making it impossible to distinguish between multiple recordings of the same title

### Reproduction Evidence

- **Commit showing reproduction:** https://github.com/NourdotSiwar/Listenarr/tree/552-show-narrator-in-match-results
- **Screenshots/logs:**  N/A — bug confirmed via code inspection
- **My findings:** LibraryImportSearchModal.vue:63–75 omits narrator. The extractNarrators() helper (fe/src/utils/searchResultHelpers.ts) already exists and handles narrator extraction. The fix pattern is already present in AddNewView.vue:654 and SearchResultCard.vue:58. No backend changes needed — narrator data is already returned in the API response.

---

## Solution Approach

### Analysis
  
Root cause: `fe/src/components/domain/audiobook/LibraryImportSearchModal.vue` lines 63–75 renders result cards with title, author, series, and ASIN but never reads the narrator field from the `SearchResult` object. The data is present — the `SearchResult` type already includes narrator — but no template code displays it in this component.

### Proposed Solution

Add a narrator line to the result card template in `LibraryImportSearchModal.vue`, using the existing `extractNarrators()` helper from `fe/src/utils/searchResultHelpers.ts` to extract the narrator string from the result object, then conditionally render `Narrated by {{ narrator }}` beneath the existing meta line.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** When a user opens the Library Import match modal and searches for an audiobook, results show title, author, and ASIN but omit the narrator. This makes it impossible to tell apart multiple recordings of the same title by different narrators. The narrator should appear on each result card.

**Match:** `fe/src/views/content/AddNewView.vue:654` already renders `Narrated by {{ book.searchResult.narrator }}` using the same `SearchResult` data shape. `fe/src/components/search/SearchResultCard.vue:58` also renders narrator via a slot. `fe/src/utils/searchResultHelpers.ts` exports`extractNarrators()` which handles all narrator data formats (string, array of strings, array of objects with `.name`).

**Plan:**

1. Modify `fe/src/components/domain/audiobook/LibraryImportSearchModal.vue` — in the result card template (lines 63–75), import `extractNarrators` from `searchResultHelpers.ts` and add a conditional narrator line after the`result-meta` span

2. Update `fe/src/__tests__/LibraryImportSearchModal.spec.ts` — add a test case asserting narrator text is rendered when the result includes narrator data

**Implement:** https://github.com/NourdotSiwar/Listenarr/tree/552-show-narrator-in-match-results *(commits will be added in Phase III)*

**Review:**

- [X] One feature/bug fix per PR (CONTRIBUTING.md)
- [X] Frontend uses 2-space indentation
- [X] No console errors or warnings
- [X] Rebased on latest `canary`
- [X] Tests added/updated
- [X] PR targets `canary` branch

**Evaluate:** Run `cd fe && npm run type-check` (no TS errors) and `npm run test:unit` (all tests pass including new narrator test). Manually confirm narrator appears on result cards in the Library Import match modal when searching for a book with known narrator data.

---

## Testing Strategy

### Unit Tests

- [X] Test case 1: Narrator renders when result includes narrator data — mocks `advancedSearch` to return a  
  result with `narrator: 'John Smith'`, mounts component, asserts `.result-narrator` contains "John Smith"
- [X] Test case 2: Result thumbnails route through protected image helper — asserts `getProtectedImageSrc` called with correct URL and placeholder, asserts `img[src]` equals the protected URL

### Integration Tests

  N/A — narrator data flows from the existing `/search` API endpoint which already
  returns narrator; no backend changes required.

### Manual Testing

  Ran local dev environment (WSL2 backend + Vite frontend). Scanned a real MP3 file,
  opened Library Import match modal, searched "Criminal". Result card displayed
  "Narrated by Kathleen Early" beneath the author line. Confirmed fix works
  end-to-end.


---

## Implementation Notes

### Week 3 Progress

Added narrator display to `LibraryImportSearchModal.vue` result cards. The narrator
  data was already present in the `SearchResult` type and returned by the API — it
  just was never rendered in this component.

Key decisions:

- Used the existing `extractNarrators()` utility from `searchResultHelpers.ts` rather than reading `result.narrator` directly — handles all narrator data formats (string, array of strings, array of `{name}` objects)

- Matched existing `.result-meta` CSS style for visual consistency (0.75rem, #888,
  ellipsis overflow)

- No backend changes needed — confirmed via Network tab that narrator data was
  already in API responses

### Week 4 Progress

Waiting for review from the maintainers, no progress to be doucmented. 

### Code Changes

- **Files modified:** 

- `fe/src/components/domain/audiobook/LibraryImportSearchModal.vue` — added `extractNarrators` import, narrator span in result card template, `.result-narrator` CSS class
- `fe/src/__tests__/LibraryImportSearchModal.spec.ts` — added narrator render test and protected image test

- **Key commits:**  [Key Commits Link](https://github.com/NourdotSiwar/Listenarr/tree/552-show-narrator-in-match-results)
- **Approach decisions:**  Copied the pattern from `AddNewView.vue:654` which already
   renders narrator in the "Add New" search flow using the same `SearchResult` shape.
  Reused `extractNarrators()` instead of accessing the field directly to stay
  consistent with how the rest of the codebase handles this.

---

## Pull Request

**PR Link:** (GitHub PR URL)[https://github.com/Listenarrs/Listenarr/pull/692]

**PR Description:** Fixes #552. The Library Import match modal showed result cards with title, author, and ASIN but never displayed the narrator, making it impossible to distinguish between multiple recordings of the same title read by different narrators.

**Maintainer Feedback:** (To be added)
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [ **Awaiting review** / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

**WSL/Windows filesystem friction:** Running .NET and Node.js tools from WSL on a Windows-mounted filesystem caused repeated issues — git config writes were blocked, npm symlinks were broken, and MSBuild couldn't set timestamps on NTFS files. Solved each individually: used git env vars (`GIT_AUTHOR_NAME` etc.) to bypass config writes, ran frontend tests from Windows PowerShell, and redirected MSBuild intermediate output to the Linux filesystem (`/tmp`) via `Directory.Build.props`.

**Backend required real audio files:** The library scanner skips zero-byte placeholder files, so the bug couldn't be reproduced with empty files. Downloaded a real MP3 from LibriVox and placed it under `/mnt/c/audiobooks/` to trigger a successful scan.

**Understanding an unfamiliar codebase in two unfamiliar languages:** This was my first time reading Vue 3 and TypeScript. Used Claude to navigate the component tree, find where narrator data flowed through the app, and identify the exact pattern to copy — the `extractNarrators()` helper in `searchResultHelpers.ts` and the narrator rendering already present in `AddNewView.vue`.

**Fresh backend instance reset configuration:** Restarting the backend wiped the root folder setting (new `.env/development` directory created). Had to re-add the root folder and re-scan before verifying the fix in the UI.

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- Mainly used Claude to be able to read the codebase, identify the problem, understand where the problem could be located, as well as assist in understanding both ReadME.md and Contribution.md
