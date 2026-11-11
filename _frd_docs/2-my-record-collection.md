# RecordCrate · My Record Collection

**Functional requirements · Draft 0.1 · Solas Dev Studio**

## Collector goal

At home, I can take ten records from my shelf, find each one in the catalogue and add it to my collection. Later, in a record shop, I can open that same collection on my phone and answer “Which Fontaines D.C. albums do I already own?”

## Scope and proposed defaults

The first release tracks ownership at album level. Each MusicBrainz release group can appear once in my collection. I can add an album without identifying its pressing, format or number of physical copies. Catalogue Search determines which albums I can add. My Record Collection owns the saved membership and makes it available wherever RecordCrate displays ownership.

This draft proposes browsing by credited artist A–Z, then by album title A–Z. Adding or removing an album affects collection membership only. I choose separately whether to keep it on a wishlist.

## Functional requirements

| ID | Capability | Required behaviour |
|---|---|---|
| MRC-01 | Add a record | From a catalogue result, I can choose **Add to collection**. After the save succeeds, the result shows **In collection** and the album appears in My Record Collection. I can carry on from the same place in the search results. |
| MRC-02 | Identify the album I own | Use the release-group identity to recognise a saved album. Repeated taps, simultaneous saves and retries produce one entry. Distinct release groups with the same title can each be owned. |
| MRC-03 | Browse my shelf | On opening the collection, show each saved album’s title, credited artist and first release year when known. Use the proposed artist/title order. An album remains visible when its year or artwork is missing. |
| MRC-04 | Find a saved record | I can filter my entire collection by artist or title, using a partial name in any letter case. Ignore surrounding spaces. Clearing the filter restores the full collection. Filtering searches saved records and remains available when MusicBrainz is unavailable, provided RecordCrate can load my collection. |
| MRC-05 | Recognise empty and unavailable states | An empty collection offers **Search the catalogue** so I can add my first record. A filter with zero matches offers **Clear filter**. While the collection loads, show progress. If loading fails, explain the problem and offer **Retry**, preserving the filter. |
| MRC-06 | Remove a record | I can select **Remove from collection** and see the album title and artist before confirming. Cancelling preserves the entry. After confirmed removal, it disappears from the collection and becomes available to add again in catalogue search. Keep any active collection filter. |
| MRC-07 | Return to the same collection | Confirmed changes persist after refresh and reopening RecordCrate. When I open my collection on another device with access to the same personal collection, I see the saved changes. |
| MRC-08 | Recover from an interrupted change | If a save or removal cannot be confirmed, identify the affected album and offer retry. Reconcile with the saved collection before reporting the outcome. Retrying a completed operation leaves the intended membership intact. |
| MRC-09 | Use the collection on phone and desktop | I can read the title and artist, filter the collection and remove a record on my phone without horizontal scrolling. Using a keyboard, I can reach the filter, choose a record’s removal action and confirm or cancel it. |

## Acceptance examples

| Check | Action | Observable outcome |
|---|---|---|
| First record · MRC-01, 05, 07 | Open an empty collection. Follow Search the catalogue, add Romance by Fontaines D.C., then open the collection on my phone. | The empty-state prompt leads to catalogue search. After saving, Romance appears once on my phone, and a fresh catalogue search shows In collection. |
| Album identity · MRC-02 | Save the same release group twice from separate tabs, then add another release group with the same title. | The collection has two entries, one for each release group. |
| Browse and filter · MRC-03, 04, 05 | With Romance by Fontaines D.C. and Disintegration by The Cure saved, enter “  FONTA  ”, then “integr”, then an unmatched title. Clear the filter. | The first two filters each find the relevant album. The third offers Clear filter. Clearing restores both albums in artist order, including any entry missing its year. |
| Catalogue unavailable · MRC-04, 05 | Make MusicBrainz unavailable while RecordCrate can still load saved records. Then separately simulate a collection load failure. | In the first case I can browse and filter my saved records. In the second I see Retry and retain my filter. |
| Correct a mistake · MRC-06, 07, 09 | With Romance in my collection and on Next buys, use the keyboard to cancel removal, then repeat and confirm. Reopen the app. | Cancel keeps the record. Confirm removes it from the collection only. Next buys still contains it, and catalogue search offers Add to collection. |
| Interrupted removal · MRC-08 | Confirm removal and simulate a lost response after the change is saved. Retry and reopen the collection. | The album stays removed. The retry resolves to that saved state and leaves every other record intact. |

## Suggested Cursor tickets

1. **Save and reopen one album.** Deliver MRC-01, MRC-02 and the persistence in MRC-07 with a minimal collection list. Coordinate with the catalogue search ownership ticket so both screens use the same saved membership. Review by tracing one add from the search result through a refresh.
2. **Browse the collection.** Deliver ordering, metadata, loading, empty and retry states. Review with an empty shelf, ten saved records and a simulated load failure.
3. **Find records on the shelf.** Deliver MRC-04 and the filter states in MRC-05. Verify partial matches against the whole collection and clear the filter on a phone.
4. **Remove an album and recover from interruptions.** Deliver MRC-06 and complete MRC-08 for both adds and removals. Demonstrate cancellation, a lost response and adding the album again. Verify wishlist membership is preserved once wishlists are available.

Check the relevant phone and keyboard behaviour from MRC-09 in each ticket. Record each reviewed change in its own commit.
