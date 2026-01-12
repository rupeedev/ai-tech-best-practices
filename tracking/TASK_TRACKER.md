# Task & Subtask Tracker

A detailed list of tasks to be completed in the project.

---

## 2025-12-14: Initial Setup

* **Task:** Configure project tracking and instruction files.
  * **Subtask:** Create `INSTRUCTIONS.md` for agent guidelines.
    * **Status:** Done (2025-12-14)
  * **Subtask:** Create `GEMINI_LOG.md` for agent action logging.
    * **Status:** Done (2025-12-14)
  * **Subtask:** Create `CHANGELOG.md` for project milestones.
    * **Status:** Done (2025-12-14)
  * **Subtask:** Create `TASK_TRACKER.md` for task management.
    * **Status:** Done (2025-12-14)

---

## 2025-12-15: GraphRAG Explorer UI Improvements

* **Task:** Redesign Advanced Search UI for better UX.
  * **Subtask:** Reorganize filters into 2-row grid layout.
    * **Status:** Done (2025-12-15)
  * **Subtask:** Convert Region, Environment, Type to auto-populated dropdowns.
    * **Status:** Done (2025-12-15)
  * **Subtask:** Add Environment filter (extracts from resource tags).
    * **Status:** Done (2025-12-15)
  * **Subtask:** Align Tag, Name/ID inputs with Search button in row 2.
    * **Status:** Done (2025-12-15)

* **Task:** Fix graph visualization bugs.
  * **Subtask:** Fix vis-network `bold` property validation warnings.
    * **Status:** Done (2025-12-15)
  * **Subtask:** Prevent graph re-execution on every keystroke (useCallback fix).
    * **Status:** Done (2025-12-15)

* **Task:** AWS MFA configuration.
  * **Subtask:** Create `.env` file for bootstrap MFA setup.
    * **Status:** Done (2025-12-15)
  * **Subtask:** Update docker-config.yaml with `infovault-mfa` profile.
    * **Status:** Done (2025-12-15)

* **Task:** Fix dropdown filters not populating.
  * **Subtask:** Investigate root cause - API response format mismatch.
    * **Status:** Done (2025-12-15)
    * **Root Cause:** API returns `[{...}]` directly, code expected `{ resources: [...] }`.
  * **Subtask:** Fix `DirectQueryMode.tsx:132` with `Array.isArray()` check.
    * **Status:** Done (2025-12-15)
  * **Subtask:** Verify dropdowns populate with Region, Environment, Resource Type.
    * **Status:** Done (2025-12-15)

* **Task:** Fix Export button to include actual resource data.
  * **Subtask:** Add state to store search results (`lastSearchResults`, `lastSearchFilters`).
    * **Status:** Done (2025-12-15)
  * **Subtask:** Create `fetchFilteredResources()` function to fetch and filter resources from API.
    * **Status:** Done (2025-12-15)
  * **Subtask:** Update `handleMultiCriteriaSearch()` to fetch and store matching resources.
    * **Status:** Done (2025-12-15)
  * **Subtask:** Update `handleExport()` to include full resource data (id, name, type, region, provider, tags, properties).
    * **Status:** Done (2025-12-15)
  * **Subtask:** Update Export button to show resource count and enable only when results exist.
    * **Status:** Done (2025-12-15)

* **Task:** Enhance Quick Queries dropdown with more common searches.
  * **Subtask:** Expand from 3 to 12 pre-built queries.
    * **Status:** Done (2025-12-15)
  * **Subtask:** Improve naming convention (short, action-oriented names).
    * **Status:** Done (2025-12-15)
  * **Subtask:** Organize queries by category (Overview, Compute, Storage, Dependencies, Audit).
    * **Status:** Done (2025-12-15)

* **Task:** Make Export button common for Quick Queries and Advanced Search.
  * **Subtask:** Add Export button to Quick Queries section.
    * **Status:** Done (2025-12-15)
  * **Subtask:** Auto-load resources when selecting a Quick Query.
    * **Status:** Done (2025-12-15)
  * **Subtask:** Update export metadata to include query source and name.
    * **Status:** Done (2025-12-15)
  * **Subtask:** Generate export filename based on query name.
    * **Status:** Done (2025-12-15)
  * **Subtask:** Separate state for Quick Query and Advanced Search results.
    * **Status:** Done (2025-12-15)
    * **Details:** Created `quickQueryResults` and `advancedSearchResults` state variables to prevent mixing results between search modes.
  * **Subtask:** Fix Quick Queries to match actual database relationships.
    * **Status:** Done (2025-12-15)
    * **Details:** Database only has EC2→VPC and EC2→Subnet relationships. Updated VPC Network Map and simplified other queries (Security Groups, S3, IAM, Multi-Region) that don't have relationships.

* **Task:** Fix Resources page Quick Filters resetting on background refresh.
  * **Subtask:** Investigate root cause - filters reset after few seconds.
    * **Status:** Done (2025-12-15)
    * **Root Cause:** `useResources` hook set `loading=true` on every 30-second refresh, causing `ResourceList` to unmount/remount and lose local filter state.
  * **Subtask:** Modify `useResources.ts` to only show loading on initial fetch.
    * **Status:** Done (2025-12-15)
    * **Details:** Added `useRef` flag (`isInitialFetch`) to track first load. Background refreshes now silently update data without affecting loading state.
  * **Subtask:** Rebuild container and verify fix.
    * **Status:** Done (2025-12-15)

* **Task:** Add CSV Export button to Resources page filter bar.
  * **Subtask:** Add Export button to the right side of filter bar.
    * **Status:** Done (2025-12-15)
  * **Subtask:** Implement CSV export function for filtered/displayed resources.
    * **Status:** Done (2025-12-15)
    * **Details:** Exports Type, Name, Resource ID, Provider, Region, Account ID, Created At, Tags columns. Properly escapes CSV special characters.
  * **Subtask:** Dynamic filename based on active filter (e.g., `resources-networking-2025-12-15.csv`).
    * **Status:** Done (2025-12-15)
  * **Subtask:** Button shows count of resources to export.
    * **Status:** Done (2025-12-15)

* **Task:** Enhance Quick Filters with dynamic counts, multi-select, and tag filter.
  * **Subtask:** Expand resource categories (7 total: Compute, Networking, Security, Storage, Database, Load Balancers, Connectivity).
    * **Status:** Done (2025-12-15)
    * **Details:** Added Database (RDS), expanded Compute (Lambda), expanded Connectivity (NatGateway, TransitGateway, VPNGateway).
  * **Subtask:** Add dynamic counts showing number of resources per category.
    * **Status:** Done (2025-12-15)
    * **Details:** Each filter button now shows count (e.g., "Compute 43"). Uses `useMemo` to calculate.
  * **Subtask:** Hide empty categories automatically.
    * **Status:** Done (2025-12-15)
  * **Subtask:** Implement multi-select support for combining filters.
    * **Status:** Done (2025-12-15)
    * **Details:** Changed from single selection to `Set<string>`. OR logic - shows resources matching ANY selected category.
  * **Subtask:** Add checkmark indicator for selected filters.
    * **Status:** Done (2025-12-15)
  * **Subtask:** Add Tagged/Untagged filter buttons.
    * **Status:** Done (2025-12-15)
    * **Details:** Two new buttons with counts: "Tagged" (resources with tags), "Untagged" (resources without tags).
  * **Subtask:** Persist multi-select and tag filter state in Saved Views.
    * **Status:** Done (2025-12-15)

* **Task:** Fix auto-refresh interval on Resources page.
  * **Subtask:** Change polling interval from 30 seconds to 5 minutes.
    * **Status:** Done (2025-12-15)
    * **Details:** Modified `useResources.ts` to poll every 300,000ms instead of 30,000ms.

* **Task:** Increase refresh interval to 1 hour and add Sync button.
  * **Subtask:** Change polling interval from 5 minutes to 1 hour.
    * **Status:** Done (2025-12-15)
    * **Details:** Modified `useResources.ts` to poll every 3,600,000ms.
  * **Subtask:** Add Sync button next to Create Resource button.
    * **Status:** Done (2025-12-15)
    * **Details:** Added purple outline "Sync" button with RefreshCw icon. Shows spinning animation while syncing.
  * **Subtask:** Add "outline" variant to Button component.
    * **Status:** Done (2025-12-15)

* **Task:** Implement URL-based routing for sidebar navigation.
  * **Subtask:** Add BrowserRouter and Routes to App.tsx.
    * **Status:** Done (2025-12-15)
    * **Details:** Routes: /dashboard, /resources, /graph-explorer, /changes, /providers, /settings
  * **Subtask:** Update Sidebar.tsx to use NavLink instead of buttons.
    * **Status:** Done (2025-12-15)
    * **Details:** NavLink provides automatic active state based on current URL.
  * **Subtask:** Update TopHeader.tsx to use useLocation hook.
    * **Status:** Done (2025-12-15)
    * **Details:** Reads current page from URL path instead of props.
  * **Subtask:** Add SPA routing support to Go backend.
    * **Status:** Done (2025-12-15)
    * **Details:** Created custom spaFileServer handler that serves index.html for client-side routes.

---

## Upcoming Tasks

* (No pending tasks)
