# RecordCrate technical design

**Solas Dev Studio · First release · Draft for review**

RecordCrate has two sources of information. MusicBrainz answers “Which albums match this search?” PostgreSQL answers “Which of those albums do I own or want?” When a collector saves an album, RecordCrate keeps enough metadata to display it again even while MusicBrainz is unavailable.

## Stack and application structure

| Decision | First-release design |
|---|---|
| Confirmed stack | React and Vite for the browser, Node.js and Express for the API, PostgreSQL accessed through pg, and Zod for runtime validation. |
| Proposed language and layout | TypeScript throughout, with client/, server/, server/migrations/ and docs/ in one repository. Keep the SQL beside the feature that uses it. |
| Local development | Run Vite and Express separately. Proxy /api from Vite to Express. Use a local PostgreSQL database and versioned SQL migrations. |
| Deployment proposal | Serve the built React assets and /api from one HTTPS origin and one Node instance. Use a persistent, backed-up PostgreSQL service. Hosting and identity vendors remain open. |

React owns form inputs, the submitted search, loaded pages and pending actions. Start with browser fetch. Express validates requests, identifies the collector, calls MusicBrainz, and executes parameterised SQL through a shared pg connection pool. Zod validates query parameters, write payloads and the subset of MusicBrainz responses the app consumes. Database constraints enforce relationships across requests.

## Private access from phone and laptop

Both devices open the same URL and sign in separately through a managed OpenID Connect provider. For the personal release, provision one permitted identity. An established authentication library handles the authorisation-code flow and callback validation. Express maps the verified issuer and subject to a collector and creates a server-side session in PostgreSQL. Each device gets its own secure, HTTP-only, same-site session cookie. Use HTTPS, expiring sessions and origin checks on state-changing requests.

Every personal query derives collector_id from that session. For a list operation, the SQL must also verify that the target list belongs to this collector. A submitted wishlist ID is only a lookup key. Expired sessions require sign-in. Refresh or opening a screen fetches current server data, so a save from the laptop is visible on the next phone visit. Launch requires a selected identity provider, registered callback URL and configured permitted identity.

## Data model

| Table | Key fields | Rule |
|---|---|---|
| collectors | id, issuer, subject | Unique (issuer, subject) maps each permitted sign-in identity to one collector. |
| albums | release_group_id UUID primary key, title, artist_credit, first_release_year nullable | One canonical saved summary per MusicBrainz release group. Preserve credited artist names and join phrases for display. |
| collection_entries | collector_id, release_group_id, added_at | Composite primary key (collector_id, release_group_id). Both identifiers reference their parent rows. |
| wishlists | id UUID primary key, collector_id, name | Foreign key to collector. Store trimmed, nonblank names. Unique index on (collector_id, lower(name)). |
| wishlist_entries | wishlist_id, release_group_id, added_at | Composite primary key (wishlist_id, release_group_id). Both identifiers reference their parent rows. |

The session store also maintains session identifiers, expiry and session data. Its exact schema follows the selected session adapter.

Buying *Romance* inserts a collection entry. Taking it off *Next buys* deletes the single matching wishlist entry. The album and its membership in *Gift ideas* survive both operations. For v1 a pressing, reissue or second physical copy does not create another ownership entry.

## Catalogue search

1. The collector submits an artist, title or a combination. React trims the input and sends a query with offset zero. Keep the submitted query separate from the editable input.
2. Express validates the query and offset. Its MusicBrainz client calls the release-group search endpoint with JSON output, a limit of 20 and the requested offset. Build the query from escaped literal search terms across release-group title and artist fields, allowing terms to match across both fields. Apply the primary-type Album filter to the whole expression. This includes album-type live releases and compilations in the draft scope.
3. One outbound scheduler spaces MusicBrainz calls at least one second apart across all users and routes, including metadata lookups. Send a descriptive User-Agent with application version and contact information. Use a timeout and surface a retryable failure when the service is busy. A second Node instance would require a shared rate limiter before deployment.
4. Map matches to album ID, title, credited artist and optional year in relevance order. Look up ownership for all returned IDs in one collector-scoped query. If that lookup fails, return ownership as unknown so the screen can offer Retry.
5. Return results and a next offset when more are available. React appends distinct IDs and ignores late responses from an earlier query. A failed next page preserves the existing results. Zero matches and service failure have separate screen states.

MusicBrainz field and request guidance: [release-group search](https://musicbrainz.org/doc/MusicBrainz_API/Search#Release_Group) · [API and rate limits](https://musicbrainz.org/doc/MusicBrainz_API).

## Saving and reading records

A save sends the release-group ID. If the album already exists, use its stored summary. For a first save, reuse a recent validated search result held briefly in server memory or, on a cache miss, look up that ID through the same MusicBrainz client and verify its type. This keeps catalogue metadata authoritative and avoids storing every search result. An unavailable lookup leaves the save retryable.

After resolving metadata, take one pg client from the pool for the whole transaction. Insert the album if absent, insert the requested membership if absent, and commit both together. Roll back on failure and release the client. Concurrent saves converge on the composite primary key. An already-saved response counts as success and preserves the original added_at. After a timeout, replaying the same membership operation is safe. Report success after confirmation.

Collection and wishlist reads join saved album summaries in PostgreSQL. Filter the entire collection by a literal case-insensitive title or artist substring using bound parameters and escaped pattern characters. Order by artist, title and ID for stable ties. Sort wishlist names A–Z and wishlist entries by newest addition first, with ID as a tie-breaker. Missing artwork or year never blocks selection.

## Proposed API surface

All paths below sit under /api and require the collector’s session.

| Method and path | Behaviour |
|---|---|
| GET /catalogue/search?q=…&offset=0 | Return album summaries, ownership state and next offset. |
| GET /collection?q=… | Return owned albums. An empty filter returns the full collection. |
| PUT or DELETE /collection/:albumId | Set or remove ownership for the signed-in collector. |
| GET /wishlists?albumId=… | List named wishlists. The optional album ID adds membership flags for the search chooser. |
| PUT /wishlists/:listId | Create a named list with a client-generated UUID. Replaying the same creation returns the existing list. A conflicting name returns a validation conflict. Renaming is outside v1. |
| GET /wishlists/:listId | Return the authorised list and its saved albums, including ownership. |
| PUT or DELETE /wishlists/:listId/albums/:albumId | Set or remove membership of that album in that list. |

For a first wishlist save, keep the generated list ID and album ID through both the create-list and add-album requests. A retry after partial success uses the same destination. Cancel before saving leaves album membership untouched. Map invalid input to 400, missing sessions to 401, unavailable or inaccessible list IDs to 404, and conflicting names to 409. Surface retryable dependency failures distinctly from empty results.

## Screens and navigation

Use three persistent navigation links: **Search**, **My Collection** and **My Wishlists**. Opening a named wishlist shows its detail view. From search, open the wishlist chooser in place so selecting a destination leaves the query, loaded pages and scroll position intact. Both a search result and a wishlist entry use the same ownership action. Display removal confirmation with the album and destination. Support keyboard completion, visible focus and a phone layout without horizontal scrolling.

## Implementation and verification

| Reviewable change | Evidence before committing |
|---|---|
| App foundation and private access | Run the client, API and migrations locally. Verify the permitted identity can enter, an unauthenticated API call is rejected and a collector cannot read or mutate a different collector’s list. |
| Search one page, then add pagination in a separate change | Exercise title-only, artist-only and mixed queries using fixtures. Verify one live search for Romance by Fontaines D.C. Check rate spacing, empty results, failure, a late response and next-page retry. |
| Save and browse collection entries | Inspect the SQL and rows in DBeaver. Save twice concurrently and verify one entry. Filter and remove a saved album during a simulated MusicBrainz outage. |
| Create lists, then add purchase actions | Exercise a lost response after list creation, save an album into two lists, purchase it and remove it from just one. Check the collection and both lists. |
| Release check | Apply migrations to the hosted database. Catalogue ten records on a laptop, sign in on a phone and verify the same records and wishlists. |

The three functional requirements documents own the detailed acceptance examples. Before hosting, choose the application host, PostgreSQL service, identity provider and session adapter. The design adopts the proposed album-only scope for v1. Including EPs or singles later changes the catalogue filter and first-save eligibility check together.
