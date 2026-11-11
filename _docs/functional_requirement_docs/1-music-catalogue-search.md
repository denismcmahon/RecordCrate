# RecordCrate · Music Catalogue Search

**Functional requirements · Draft 0.1 · Solas Dev Studio**

## Collector goal

Standing in a record shop, I can search for an album, recognise the right result, see whether I own it, and save it to my collection or a named wishlist. At home, I can use the same search to catalogue the next record on my shelf.

## Scope and proposed defaults

Search covers albums in the MusicBrainz catalogue. A search result represents one album (a MusicBrainz release group). A vinyl edition, CD edition or reissue of the same album belongs to that result. The collector can save the album without identifying a pressing. For this draft, include groups with the primary type Album, including live albums and compilations of that type. Singles and EPs are a future scope decision.

Use one search field with an explicit Search action. Show 20 matches at a time, most relevant first, with a Load more action. These are proposed first-release choices. Collection editing and wishlist management have their own requirements. This document covers what the collector experiences when using either feature from search.

## Functional requirements

| ID | Capability | Required behaviour |
|---|---|---|
| MCS-01 | Submit a search | Accept an artist name, album title, or artist and title together. Support the Search button and Enter. Ignore surrounding spaces. A blank submission prompts the collector to enter an artist or album and sends no search. |
| MCS-02 | Identify an album | Show the album title and credited artist for every result, and the first release year when known. Group editions by release-group identity. Distinct albums with the same title remain separate. Missing year or artwork never prevents selection. |
| MCS-03 | Browse matches | Show up to 20 results initially. When more are available, Load more appends the next matches without repeating albums already displayed. A new search starts at the first page. |
| MCS-04 | Understand search status | Before the first search, explain what to enter. While a search is running, indicate progress. A successful search with no matches offers a suggestion to try the artist or album title alone. A service or connection failure offers Retry and retains the query. Only the most recently submitted query can update the current results. |
| MCS-05 | Recognise ownership | Mark each owned album In collection. For an album the collector can add, show Add to collection. Identify ownership by release group so equally titled albums by different artists have independent states. If ownership cannot be checked, show that it is unavailable and offer a retry. |
| MCS-06 | Add to collection | Save the selected album directly from the results. Once saved, show In collection in place of the add action. Keep the query, results and position. Repeated taps or retries can create only one collection entry for that album. |
| MCS-07 | Add to a wishlist | Let the collector choose a named wishlist, showing which lists already contain the album. If no list exists, offer to create the first one and continue with the selected album. On success, confirm the destination by name. Cancelling leaves membership unchanged. Each list can contain the album once. |
| MCS-08 | Recover from a failed action | If a save fails, show which action failed, preserve the result and query, and offer a retry. Show a success confirmation only after the save is confirmed. If loading more matches fails, keep the matches already displayed. |
| MCS-09 | Use search on phone and desktop | On a phone, the title, artist, ownership state and save actions are accessible without horizontal scrolling. With a keyboard, the collector can submit a query, choose a result action and complete or cancel wishlist selection. |

## Acceptance examples

| Check | Action | Observable outcome |
|---|---|---|
| Artist, title and combined search | With a catalogue fixture containing Romance by Fontaines D.C., submit “Fontaines D.C.”, “Romance” and “Fontaines D.C. Romance” separately. | Each result set includes the matching album. Two editions in the same release group yield one entry. |
| Same title, different album | Return two release groups with the same title. The collection contains only the first. | Both appear, and only the first shows In collection. |
| Empty, unavailable and no matches | Submit spaces, submit a query with zero matches, then simulate a failed catalogue lookup. | The collector sees an input prompt, a no-matches message and a retry action respectively. Retrying uses the same query. |
| More matches and overlapping searches | Return 21 distinct matches. Load more, then submit a different query while the previous request is pending. | The first page contains 20 matches. The new query starts at page one. A late response cannot append old albums to the new results. With no intervening search, Load more displays the 21st album exactly once. |
| Ownership persists | Tap Add to collection twice for Romance, then reopen RecordCrate and search again. | There is one saved entry. The result shows In collection. |
| Wishlist selection | Choose Add to wishlist, cancel, then reopen the chooser and select Next buys. Repeat for the same list. | Cancelling saves nothing. A successful save identifies Next buys. The album appears there once and collection ownership is unchanged. |
| First wishlist and failed save | Start with no wishlists. Create Next buys from the chooser. Simulate a failed album save, then retry. | The collector can finish adding the original album without searching again. The confirmation appears after successful retry and the list contains one copy. |

## Suggested Cursor tickets

1. **Search and recognise an album.** Implement one submitted query, the first results page and the loading, empty and error states. Demonstrate with an artist search and a title search.
2. **Browse further results.** Add Load more and verify that late responses and failed page loads preserve the correct results.
3. **See and update ownership.** Once collection storage is available, show ownership and support saving from a result. Verify with a repeated tap and a fresh search.
4. **Save to a named wishlist.** Once wishlist storage is available, add destination selection, first-list creation, cancellation and retry.

Review each ticket on phone and desktop. Before committing, trace one user action through the changed code and demonstrate its acceptance examples.
