# Changelog

A summary of major changes and milestones in the project.

---

### December 2025

* **2025-12-14:** Project initialization and agent setup.
  * Established interactive logging (`GEMINI_LOG.md`).
  * Created agent interaction guidelines (`INSTRUCTIONS.md`).
  * Set up changelog (`CHANGELOG.md`) and task tracker (`TASK_TRACKER.md`).
* **2025-12-15:** Advanced Search UI redesign and bug fixes.
  * Redesigned Advanced Multi-Criteria Search with professional 2-row grid layout.
  * Row 1: Region, Environment, Resource Type (auto-populated dropdowns).
  * Row 2: Tag Value, Name/Resource ID (text inputs) + Search button.
  * Added Environment filter that extracts values from resource tags.
  * Fixed vis-network `bold` property validation warnings in graph visualization.
  * Fixed graph re-execution on keystroke issue using `useCallback` memoization.
  * Updated AWS profile configuration to `infovault-mfa` for MFA authentication.
  * Created `.env` file for bootstrap MFA setup.
  * **Bug Fix:** Fixed dropdown filters not populating - API returns direct array `[{...}]` but code expected `{ resources: [...] }`. Added `Array.isArray()` check to handle both formats.
  * **Bug Fix:** Fixed Export button to include actual resource data instead of just query/filters. Export now includes full resource information (id, name, type, region, provider, tags, properties).
  * **Enhancement:** Expanded Quick Queries from 3 to 12 common searches with improved naming convention.
    * Overview: All Resources, Resource Count by Type
    * Compute/Network: EC2 Instances, VPC Network Map, Security Groups
    * Storage: S3 Buckets
    * Dependencies: Dependency Chains, Cross-Service Dependencies
    * Audit: Recently Discovered, Untagged Resources
    * Regional: Multi-Region Overview, IAM Resources
  * **Enhancement:** Made Export button common for both Quick Queries and Advanced Search.
    * Export button now appears in Quick Queries section
    * Auto-loads resources when selecting a Quick Query
    * Export filename includes query name (e.g., `graphrag-ec2-instances-*.json`)
    * Export metadata includes source (quick/advanced) and query name
  * **Bug Fix:** Separated Export button state for Quick Queries and Advanced Search.
    * Each section now tracks its own results independently
    * Advanced Search Export only shows count after running a search
    * Quick Queries Export only shows Quick Query results
    * Prevents confusion from mixing results between search modes
  * **Bug Fix:** Fixed Quick Queries to match actual database relationships.
    * VPC Network Map: Updated to use EC2→VPC and EC2→Subnet relationships (was expecting non-existent Subnet→VPC)
    * Security Groups, S3 Buckets, IAM Resources, Multi-Region: Simplified to show resources without expecting relationships that don't exist
    * Queries now correctly display graph visualizations based on actual Neo4j data model
  * **Bug Fix:** Fixed Resources page Quick Filters resetting on background refresh.
    * Root cause: `useResources` hook set `loading=true` on every 30-second poll, causing `ResourceList` component to unmount/remount and lose filter state
    * Fix: Added `useRef` flag to only show loading spinner on initial fetch; background refreshes now silently update data
    * File modified: `agentic-infra-ui/src/hooks/useResources.ts`
  * **Feature:** Added CSV Export button to Resources page filter bar.
    * Green "Export (N)" button on the right side of filter bar
    * Exports currently displayed/filtered resources to CSV format
    * Columns: Type, Name, Resource ID, Provider, Region, Account ID, Created At, Tags
    * Dynamic filename includes active filter name (e.g., `resources-networking-2025-12-15.csv`)
    * File modified: `agentic-infra-ui/src/components/ResourceList.tsx`
  * **Feature:** Enhanced Quick Filters with dynamic counts, multi-select, and tag filtering.
    * Expanded to 7 categories: Compute (EC2, Lambda), Networking, Security, Storage, Database (RDS), Load Balancers, Connectivity (EIP, NAT, Transit, VPN Gateways)
    * Dynamic counts: Each filter shows resource count (e.g., "Compute 43")
    * Multi-select: Click multiple filters to combine (OR logic)
    * Checkmark indicator on selected filters
    * Empty categories automatically hidden
    * Added Tagged/Untagged filter buttons with counts
    * Saved Views now persist multi-select and tag filter state
    * File modified: `agentic-infra-ui/src/components/ResourceList.tsx`
  * **Fix:** Changed auto-refresh interval from 30 seconds to 5 minutes.
    * Reduces unnecessary API calls and UI disruption
    * File modified: `agentic-infra-ui/src/hooks/useResources.ts`
  * **Feature:** Increased refresh interval to 1 hour and added Sync button.
    * Auto-refresh now polls every 1 hour instead of 5 minutes
    * Added "Sync" button next to "Create Resource" for manual refresh
    * Button shows spinning animation while syncing
    * Added "outline" variant to Button component
    * Files modified: `agentic-infra-ui/src/hooks/useResources.ts`, `agentic-infra-ui/src/components/Resources.tsx`, `agentic-infra-ui/src/components/ui/Button.tsx`
  * **Feature:** Implemented URL-based routing for sidebar navigation.
    * Routes: `/dashboard`, `/resources`, `/graph-explorer`, `/changes`, `/providers`, `/settings`
    * Sidebar now uses `NavLink` with automatic active state highlighting
    * Browser back/forward navigation works correctly
    * Direct URL access supported (e.g., `localhost:8080/resources`)
    * Added SPA routing support to Go backend (serves index.html for client-side routes)
    * Files modified: `agentic-infra-ui/src/App.tsx`, `agentic-infra-ui/src/components/Sidebar.tsx`, `agentic-infra-ui/src/components/TopHeader.tsx`, `agentic-infra/internal/api/server.go`
