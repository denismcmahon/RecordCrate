# RecordCrate · My Wishlists

**Functional requirements · Draft 0.1 · Solas Dev Studio**

## Collector goal

At home, I find *Romance* by Fontaines D.C. and save it to *Next buys*. In a record shop, I can open that list on my phone, recognise the album, and add it to my collection if I buy it. I decide whether to take it off *Next buys*.

## Scope and proposed defaults

The first release supports personal, named wishlists. Catalogue Search supplies the selected album. My Wishlists records which lists contain it. My Record Collection records ownership. A wishlist entry represents one MusicBrainz release group, so the collector can save an album without choosing a particular edition.

Proposed defaults for review: start with no pre-created lists, require distinct list names within the collector’s account or personal workspace, show list names A–Z, and show the most recently added albums first within each list. Renaming and deleting whole lists are deferred from this draft. Removing individual albums is included.

## Functional requirements

| ID | Capability | Required behaviour |
|---|---|---|
| MWL-01 | Create a named list | Offer **Create wishlist** from My Wishlists, including when no lists exist. Trim surrounding spaces. A blank name or a name already used by this collector, ignoring case, prompts for a different name. A successful creation produces one empty list. |
| MWL-02 | Save from catalogue search | **Add to wishlist** opens a chooser for the selected album. Show available lists by name and identify those already containing the album. If no list exists, allow the collector to create one and continue with the same album. Confirm the destination by name after a successful save. Preserve the search query and position on return. Cancelling saves no album. |
| MWL-03 | Save to more than one list | A release group can appear once in each of several lists. Repeated taps, concurrent saves and retries for the same album and list produce one entry. Different release groups with the same title can each be saved. Saving an album to a list leaves ownership unchanged. |
| MWL-04 | Browse saved albums | Opening My Wishlists shows the collector’s named lists. Opening a list shows its name and saved albums, with title, credited artist and first release year when known. The empty-list view offers **Search the catalogue**. Missing year or artwork leaves the album selectable. Browsing saved albums remains available during a MusicBrainz outage when RecordCrate can load the list. |
| MWL-05 | Record a purchase | From a wishlist entry, offer **Add to collection** for an unowned album and show **In collection** for an owned album. Apply the collection’s duplicate and save-confirmation rules. If ownership cannot be checked, indicate that and offer Retry. After a purchase, keep the album on each wishlist until the collector explicitly removes it. |
| MWL-06 | Remove an album | **Remove from this wishlist** identifies the album and list before confirmation. Cancelling keeps the entry. Confirming removes it only from that list. Removing its last album leaves the named list available for future use. |
| MWL-07 | Return on another device | Confirmed lists and memberships persist after refresh and reopening. Opening the same personal workspace on another device shows the saved changes. |
| MWL-08 | Recover from a failure | Distinguish loading, empty and unavailable lists. A failed load offers Retry. If a change cannot be confirmed, retain the album and destination in context, offer retry, and confirm success only when the saved state is known. If list creation succeeded but the album save failed, retry against the list already created. |
| MWL-09 | Use on phone and desktop | On a phone the collector can read the list name, album title and artist and reach every action without horizontal scrolling. With a keyboard the collector can create a list, choose a destination, browse, add to collection, and confirm or cancel removal. |

## Acceptance examples

| Check | Action | Observable outcome |
|---|---|---|
| First save from search · MWL-01, 02 | With no wishlists, search for Romance, choose Add to wishlist, create Next buys and save. | Confirmation names Next buys. The original search position is preserved. Opening the list shows Romance by Fontaines D.C. once. |
| Names and cancellation · MWL-01, 02 | With Next buys present, try creating a whitespace-only name and then “ next BUYS ”. Open the save chooser for another album and cancel. | Each invalid name has an explanation. No extra list appears, and cancelling adds no album. |
| Membership and identity · MWL-02, 03 | Save the same release group to Next buys from two tabs, then save it to Gift ideas. Reopen the chooser. | Each list has one copy and both are marked in the chooser. A separate release group with the same title can still be added. |
| Browse in the shop · MWL-04, 07, 09 | Save two albums on a laptop. Open the same list on a phone while MusicBrainz is unavailable and RecordCrate is reachable. | Both albums are readable, newest addition first. Missing artwork or year does not block browsing. |
| Purchase and removal · MWL-05, 06 | With Romance on Next buys and Gift ideas, add it to the collection from Next buys. Cancel a removal, then confirm removal from Next buys. | After purchase, Romance shows In collection and remains on both lists. Cancelling changes nothing. After removal it remains owned and on Gift ideas. If Next buys is now empty, it still opens. |
| Partial save · MWL-08 | Create a first list from search, interrupt the album save, and retry. Repeat with a lost response after a successful save. | Both paths finish with one named list and one album entry. The collector can complete either path without repeating the catalogue search. |

## Suggested Cursor tickets

1. **Create and reopen a named wishlist.** Deliver name validation, the list overview, persistence and the empty-list view. Review with a fresh collection of lists and a duplicate name.
2. **Save one search result to a chosen list.** Complete the chooser shared with MCS-07, including first-list creation, cancellation, duplicate handling and a failed save. Demonstrate the flow from an actual search result.
3. **Browse a list and remove an entry.** Show saved metadata and ordering, handle unavailable data, and demonstrate removing an album from just one of two lists.
4. **Record a purchase from a wishlist.** Reuse the ownership behaviour from MRC-01 and MRC-02. Demonstrate purchase, refresh and optional removal as separate collector actions.

For each ticket, exercise the relevant acceptance examples on phone and desktop, review the changed code, and commit the verified behaviour.
