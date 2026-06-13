# Contribution 552: When matching existing audiobooks, search results should show the narrator

**Contribution Number:** 552

**Student:** Nour Siwar

**Issue:** [GitHub issue link](https://github.com/Listenarrs/Listenarr/issues/552)

**Status:** Phase II - Complete

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

- [ ] One feature/bug fix per PR (CONTRIBUTING.md)
- [ ] Frontend uses 2-space indentation
- [ ] No console errors or warnings
- [ ] Rebased on latest `canary`
- [ ] Tests added/updated
- [ ] PR targets `canary` branch

**Evaluate:** Run `cd fe && npm run type-check` (no TS errors) and `npm run test:unit` (all tests pass including new narrator test). Manually confirm narrator appears on result cards in the Library Import match modal when searching for a book with known narrator data.

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
