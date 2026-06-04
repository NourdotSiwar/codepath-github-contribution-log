# Contribution 552: When matching existing audiobooks, search results should show the narrator

**Contribution Number:** 552

**Student:** Nour Siwar

**Issue:** [GitHub issue link](https://github.com/Listenarrs/Listenarr/issues/552)

**Status:** Phase I - Complete

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

[Notes on setting up your local development environment - challenges you faced, how you solved them]

### Steps to Reproduce

1. [Step 1]
2. [Step 2]
3. [Observed result]

### Reproduction Evidence

- **Commit showing reproduction:** [Link to commit in your fork]
- **Screenshots/logs:** [If applicable]
- **My findings:** [What you discovered during reproduction]

---

## Solution Approach

### Analysis

[Your analysis of the root cause - what's causing the issue?]

### Proposed Solution

[High-level description of your fix approach]

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** [Restate the problem]

**Match:** [What similar patterns/solutions exist in the codebase?]

**Plan:** [Step-by-step implementation plan]
1. [Modify file X to do Y]
2. [Add function Z]
3. [Update tests]

**Implement:** [Link to your branch/commits as you work]

**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]

**Evaluate:** [How will you verify it works?]

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
