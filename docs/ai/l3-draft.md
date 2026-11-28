Tool: Claude Opus 5.5 (medium effort)
Date: 2026-11-28
Prompt: docs/ai/l3-prompt.txt

# Spec: find movies for a child by rating

Source: `intent.md` (authors 6810301003 and 6810301037, for the customer proxy: a parent of an 8-year-old). Status: draft.
Scope: the Rotten Potatoes movie catalog domain classes (`Movie`, `Rating`, `MovieCatalog` from L2). There is no web page, no database and no login. All behaviour is checked through the domain classes in plain Java.

---

## 1. Functional requirements

Each requirement can be checked by a unit test against the domain classes.

| ID | Requirement | How to test it |
|----|-------------|----------------|
| **REQ-1** | The system shall recognise exactly five ratings: `G`, `PG`, `PG-13`, `R`, `NC-17`. | Enumerate the supported ratings and assert the set is exactly these five, no more and no fewer. |
| **REQ-2** | Given one chosen rating, `MovieCatalog` shall return only the movies whose rating equals that rating. | Build a catalog with movies of mixed ratings, filter by `PG`, and assert every returned movie has rating `PG`. |
| **REQ-3** | The filtered result shall include every movie in the catalog that has the chosen rating. | Build a catalog with 3 `PG` movies among others, filter by `PG`, and assert all 3 are returned. |
| **REQ-4** | Matching shall be exact: filtering by `PG` shall not return `PG-13` movies, and filtering by `PG-13` shall not return `PG` movies. | Build a catalog with one `PG` and one `PG-13` movie. Filter by each and assert only the exact match comes back. |
| **REQ-5** | The filtered result shall be ordered by release date, newest first. | Add `PG` movies released 2019, 2023 and 2021 in that order. Filter by `PG` and assert the order is 2023, 2021, 2019. |
| **REQ-6** | Choosing a value that is not one of the five ratings shall be rejected, and no movie list shall be returned for it. | Attempt to filter by `"PG-15"` (or an equivalent invalid value) and assert it is rejected. |
| **REQ-7** | Filtering shall need no login, user identity or personal data. | The filter operation's inputs are only the catalog and the rating. A test calls it without any user or session object. |
| **REQ-8** | Filtering shall run with in-memory data only (no database, no web layer). | The tests build `MovieCatalog` from in-memory `Movie` objects and pass with no database or server running. |

> **Not yet specified on purpose:** what the parent sees when no movie has the chosen rating (Open question 1), and whether several ratings can be chosen at once (Open question 2). See section 4.

---

## 2. User stories

### Story 1: Filter by one rating

**As a** parent of an 8-year-old,
**So that** I can find a suitable movie quickly without opening each movie to check its rating,
**I want** to pick one rating and see only the movies with that rating.

Acceptance criteria:
- AC-1.1: After I pick a rating, every movie shown has exactly that rating (REQ-2, REQ-4).
- AC-1.2: No movie with the chosen rating is left out (REQ-3).
- AC-1.3: I can only pick from G, PG, PG-13, R and NC-17. Any other value is rejected (REQ-1, REQ-6).

### Story 2: Newest movies first

**As a** parent of an 8-year-old,
**So that** I see recent movies my child is likely to want without scrolling through old ones,
**I want** the filtered movies to be shown newest first.

Acceptance criteria:
- AC-2.1: The filtered list is ordered by release date, newest first (REQ-5).
- AC-2.2: The ordering applies only to the filtered movies. Movies with other ratings do not appear (REQ-2).

### Story 3: Any moviegoer can filter without signing in

**As a** moviegoer (not only a parent),
**So that** I can narrow a long catalog to the kind of movie I want without creating an account,
**I want** to use the same rating filter without logging in or giving any personal data.

Acceptance criteria:
- AC-3.1: Filtering works with no login, identity or personal data (REQ-7).
- AC-3.2: Filtering gives the same result for every user who picks the same rating on the same catalog (REQ-2, REQ-5, REQ-7).

---

## 3. Gherkin scenarios for Story 1

```gherkin
Feature: Filter the movie catalog by one rating
  As a parent of an 8-year-old
  So that I can find a suitable movie quickly
  I want to pick one rating and see only the movies with that rating

  Background:
    Given the movie catalog contains these movies:
      | title          | rating | release date |
      | Paddington 2   | PG     | 2017-11-10   |
      | Turning Red    | PG     | 2022-03-11   |
      | Frozen         | PG     | 2013-11-27   |
      | Spider-Verse   | PG     | 2018-12-14   |
      | Jumanji        | PG-13  | 2017-12-20   |
      | Toy Story      | G      | 1995-11-22   |
      | The Matrix     | R      | 1999-03-31   |

  # Happy path
  Scenario: Parent picks PG and sees only PG movies, newest first
    When the parent picks the rating "PG"
    Then the parent sees exactly these movies in this order:
      | title          |
      | Turning Red    |
      | Spider-Verse   |
      | Paddington 2   |
      | Frozen         |
    And the parent does not see "Jumanji"

  # Sad path: value outside the allowed ratings
  Scenario: Parent picks a value that is not a valid rating
    When the parent picks the rating "PG-15"
    Then the choice is rejected as an unknown rating
    And no movie list is shown

  # Sad path: no movie has the chosen rating (pending Open question 1)
  @pending-OQ1
  Scenario: Parent picks a valid rating that no movie has
    When the parent picks the rating "NC-17"
    Then the parent sees no movies
    # What the parent sees in this case is still Open question 1.
    # This scenario only checks that no non-NC-17 movie leaks into the result.
```

---

## 4. Assumptions and areas of concern

### Assumptions (not stated in the intent)

| ID | Assumption |
|----|------------|
| A-1 | Each `Movie` has exactly one rating and a release date. "Newest" means the latest release date. |
| A-2 | Rating matching is exact on the rating's value. The rating is chosen from the five allowed values (e.g. an enum in `Rating`), not typed as free text. So case handling ("pg" vs "PG") does not arise. If free text is allowed, this needs a decision. |
| A-3 | "Sees only the movies" means `MovieCatalog` returns a list. Display is out of scope because there is no web page yet. |
| A-4 | Filtering does not change or reorder the underlying catalog. It returns a new result. |
| A-5 | An invalid rating is rejected (e.g. an exception or a failed parse into `Rating`) rather than returning an empty list. This is inferred from the "exactly G, PG, PG-13, R and NC-17" constraint. |
| A-6 | Until Open question 1 is answered, a valid rating with no matching movies returns an empty list. The `@pending-OQ1` scenario assumes only this. |
| A-7 | Only one rating can be chosen per filter, as the proposed outcome says "picks one rating". Multi-select (Open question 2) is out of scope until answered. |
| A-8 | The movie titles and dates in the Gherkin are sample test data, not real catalog content. |
| A-9 | Story 2 and Story 3 come from the proposed outcome ("newest first") and the affected users ("then every moviegoer"). They are not separate requests from the customer. |

### Areas of concern

| ID | Concern |
|----|---------|
| C-1 | **Open question 1 is unanswered:** the empty-result behaviour (blank list, message, suggestion of other ratings?) is undecided. The parent's experience in this case is unspecified. |
| C-2 | **Open question 2 is unanswered:** if several ratings become selectable (e.g. G *and* PG for an 8-year-old), REQ-2, REQ-4 and the filter's signature change. Deciding this late could mean rework. |
| C-3 | **Tie-breaking is undefined:** two movies with the same release date have no defined order, so REQ-5 tests could be flaky. A secondary order (e.g. by title) may be needed. |
| C-4 | **Missing data:** the intent does not say what happens to a movie with no release date, or with no rating, in the L2 classes. They might be skipped, sorted last, or cause an error. |
| C-5 | **The `PG` / `PG-13` prefix trap:** a naive "starts with" or "contains" comparison would wrongly mix them. REQ-4 guards this. |
| C-6 | **Fit with existing L2 classes:** `Movie`, `Rating` and `MovieCatalog` already exist. Their current fields (does `Movie` have a release date?) and the representation of `Rating` must be checked before this spec can be implemented. |
| C-7 | **The problem is only partly solved:** a single rating may not match what the parent actually wants for an 8-year-old (G *or* PG). This ties to C-2 and should be confirmed with the customer proxy. |
| C-8 | **No performance target:** the intent says the catalog is "long" but gives no size or speed goal, so no non-functional requirement is written. |
