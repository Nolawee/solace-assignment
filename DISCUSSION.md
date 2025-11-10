# Discussion & Implementation Notes

## Why is there one PR and two commits?

I originally was a bit on autopilot and merged my PRs into main in my original repo, just due to my current workflow at my job. When I remembered we wanted a link to all my open PRs and for main to be the original zip file so it's easy to review, I decided to finish the rest of the project locally and create a new repo with a clean commit history and one PR that details my main changes.

**Link to original repo's PRs:** [http://github.com/Nolawee/solace-candidate-assignment/pulls?q=is%3Apr+is%3Aclosed]

---

## Improvements & Next Steps

These are improvements I'd like to make if this were a more fleshed-out project:

### 1. **Front-End Pagination**
I have cursor-based pagination fully configured on the backend and tested it with page sizes of 5, but ran out of time to implement the actual pagination UI on the frontend. The API already returns `pageInfo` with `hasNextPage` and `nextCursor`, so adding "Load More" or infinite scroll would be straightforward.

**What's needed:**
- "Load More" button or infinite scroll implementation
- State management for accumulated results
- Loading states during pagination requests
- Cursor tracking in URL params (optional)

### 2. **Better Use of React Hooks**
I primarily used `useEffect` here, which works fine for this smaller dataset, but for a production app I could take advantage of:
- **`useMemo`** - Memoize filtered/sorted data to prevent unnecessary recalculations
- **`useCallback`** - Memoize event handlers to prevent child component re-renders
- **`useTransition`** - For non-blocking UI updates during search/filter operations
- **Custom hooks** - Extract search logic, filter logic, and API calls into reusable hooks like `useAdvocateSearch()` or `useDebounce()`

### 3. **Better Use of Tailwind**
While I used Tailwind utilities throughout, there are opportunities to:
- Extract common patterns into custom utility classes or components
- Use `@apply` for repeated style combinations
- Leverage Tailwind's built-in responsive utilities more consistently
- Utilize the full color palette instead of hardcoded hex values
- Create a custom Tailwind theme config with the Solace brand colors

### 4. **Individual Advocate Modals/Pages**
Currently, advocates are displayed as cards in a list. For a better UX, I'd add:
- Detailed advocate pages (`/advocates/[id]`) with full bio, credentials, availability
- Modal overlay for quick view without navigation
- More specialty details and filtering by specialty tags
- Reviews/ratings system
- Contact form integration

### 5. **Proper Database Migrations**
When updating the schema, instead of taking advantage of Drizzle's migration system and altering my existing schema, I dropped my tables in the terminal and created new ones. In a production environment, I would:
- Use `drizzle-kit generate` to create migration files
- Review and test migrations in a staging environment
- Run migrations with proper rollback strategies
- Maintain migration history in version control
- Never drop tables with production data!

### 6. **Unit and Integration Testing**
I didn't implement tests due to time constraints, but for a production application, comprehensive testing is critical:

**Why Testing Matters:**
- **Confidence in refactoring** - Can safely modify code without breaking existing features
- **Regression prevention** - Catch bugs before they reach production
- **Documentation** - Tests serve as living documentation of expected behavior
- **Faster debugging** - Failing tests pinpoint exactly what broke

**What I'd test:**

**Backend (Unit & Integration):**
- API route handlers (`/api/advocates`, `/api/seed`)
- Search query construction with various inputs (empty, special characters, SQL injection attempts)
- Pagination logic and cursor generation
- Filter combinations (degree + search + sort)
- Database transactions and rollback behavior
- Full-text search accuracy and ranking
- Edge cases (no results, malformed cursors, invalid filters)

**Frontend (Unit & Integration):**
- Component rendering (AdvocateCard, Search, AdvocateList)
- Search input debouncing behavior
- Filter state management and URL synchronization
- Reset button enabling/disabling logic
- API error handling and loading states
- Accessibility features (keyboard navigation, screen readers)
- Responsive design breakpoints

**E2E Testing:**
- Complete user flows (search → filter → sort → reset)
- Deep link handling (opening a URL with filters pre-applied)
- Mobile vs. desktop experiences
- Browser compatibility

**Testing Tools I'd Use:**
- **Jest** + **React Testing Library** for frontend unit/integration tests
- **Playwright** or **Cypress** for E2E tests
- **MSW (Mock Service Worker)** for mocking API calls in frontend tests
- **Drizzle's testing utilities** for database tests with in-memory SQLite

---

## Technical Decisions & Trade-offs

### Database Design
**Decision:** Normalized schema with separate `specialties` table instead of JSONB array.

**Rationale:**
- Enables full-text search on specialties independently
- Prevents data duplication (specialties stored once)
- Allows for future specialty management (descriptions, categories, etc.)
- Better query performance with proper indexes

**Trade-off:** More complex queries with joins, but better scalability

### Full-Text Search
**Decision:** PostgreSQL's native `tsvector` with `websearch_to_tsquery` instead of external search services (Algolia, ElasticSearch).

**Rationale:**
- No additional infrastructure required
- Lower latency (no network hop)
- Sufficient for this dataset size
- Built-in ranking and relevance scoring

**Trade-off:** For very large datasets (millions of records), a dedicated search engine would be better

### Cursor-Based Pagination
**Decision:** Cursor pagination with composite key (`yearsOfExperience_id`) instead of offset pagination.

**Rationale:**
- Consistent results even when data changes
- Better performance on large datasets (no expensive OFFSET queries)
- Prevents duplicate/missing records during pagination

**Trade-off:** Can't jump to arbitrary pages (page 5, page 10), but better for infinite scroll patterns

### Client-Side vs. Server-Side Rendering
**Decision:** Client-side rendering with `"use client"` and API routes instead of Next.js Server Components.

**Rationale:**
- Interactive search/filter requires client-side state
- Real-time updates without full page reloads
- More familiar React patterns

**Trade-off:** Slower initial load, no SEO benefits (could be mitigated with server-side rendering + hydration)

---

## Questions & Clarifications

If I were to continue this project, I'd want to clarify:

1. **User Authentication** - Should advocates have login portals? Should clients be able to save favorites?
2. **Availability Scheduling** - Should advocates indicate their availability for calls/sessions?
3. **Search Ranking** - What's the priority order? (exact name match > specialty match > city match?)
4. **Specialty Management** - Should there be an admin panel to manage the specialty list?
5. **Analytics** - What metrics do we want to track? (popular searches, most contacted advocates, etc.)

---

## Conclusion

This was a very fun project and I can't wait to hear feedback!
