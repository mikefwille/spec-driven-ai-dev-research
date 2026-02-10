# Session Initialization - Marqet Project

Welcome to a new Marqet development session.

## 1. Git Branch Management & Jira Ticket Creation

**Run this single command to check branch status efficiently:**

```bash
BRANCH=$(git branch --show-current) && echo "Branch: $BRANCH" && if [ "$BRANCH" != "main" ]; then echo "Unmerged commits:" && git log origin/main..HEAD --oneline 2>/dev/null || echo "(none)"; fi
```

### Quick Decision Logic:

1. **If on `main` with clean status** → Skip to "Ready for New Work" section below (no further git checks needed)

2. **If on a feature branch WITH unmerged commits** → Ask user:
   - "You're on `[branch-name]` with X unmerged commits. Continue here or create new branch?"

3. **If on a feature branch with NO unmerged commits** → Auto-switch to main:
   ```bash
   git checkout main && git pull origin main
   ```
   Then proceed to "Ready for New Work"

### Branch Strategy:

- **main** - Production-ready code
- **Feature branches** - Use format: `feat/MAR-XX/description` or `fix/MAR-XX/description`

---

### Ready for New Work (on main or user wants new branch):

We'll create a Jira ticket and feature branch together:

**Step 1: Check for Existing Tickets**

- Get current user info from Atlassian MCP (accountId, email, displayName)
- Ask user: "Do you have an existing Jira ticket for this work? (Provide ticket number like MAR-87, or say 'new' to create one)"
- Search Jira to show current user's open tickets for reference
- If multiple tickets in backlog, show them so user can pick one

**Step 2: Create or Use Ticket**

**Option A: Using Existing Ticket**

- User provides ticket number (e.g., MAR-87)
- Verify ticket exists and is assigned to current user
- Transition to "In Progress" if not already
- Get ticket summary for branch name suggestion

**Option B: Creating New Ticket**

- Ask user: "What are you working on? (Brief description)"
- Create ticket in MAR project with description
- Assign to current user (using accountId from atlassianUserInfo)
- Set status to "In Progress"
- Get ticket number (e.g., MAR-87)

**Step 3: Create Branch with Ticket Number**

```bash
# Format: feat/MAR-XX/description or fix/MAR-XX/description
git checkout -b feat/MAR-87/your-feature-name
```

**Step 4: Link Branch to Ticket**

- Add comment to Jira ticket with branch name
- Document what will be implemented
- Add link to commit history

### Branch Naming Convention:

```bash
# ✅ CORRECT - Includes ticket number
feat/MAR-87/form-playbook-integration
fix/MAR-88/oauth-token-refresh
chore/MAR-89/update-dependencies

# ⚠️ ACCEPTABLE - Auto-create ticket
feat/form-creator-part-2    # Will create MAR-XX automatically
fix/user-enrollment-loading  # Will create MAR-XX automatically
```

### If Already on Feature Branch:

Check if ticket exists:

- Search Jira for branch name or recent work
- If no ticket found, create one and link it
- Update ticket status to "In Progress"

**Jira Integration:**

- All work requires a Jira ticket
- Tickets auto-assigned to current authenticated user
- Board: https://marq.atlassian.net/jira/software/projects/MAR/boards/247
- Config: `.jira-config.json`
- Uses Atlassian MCP for authentication and user lookup

## 2. Development Standards

### Performance-First Mindset

**Key Principles:**

- **Build success = merge ready** - If it builds and runs, it's functional
- **TypeScript warnings ≠ blockers** - Address when convenient, not before merge
- **Focus on active features** - Don't spend time on unused code paths
- **Incremental improvements** - Ship working code, refine later

### 🚨 File Size Limits & Organization

**CRITICAL: Keep files under 1000 lines**

| File Size      | Action Required                  |
| -------------- | -------------------------------- |
| < 300 lines    | ✅ Ideal - well-scoped file      |
| 300-500 lines  | ⚠️ Good - monitor complexity     |
| 500-800 lines  | ⚠️ Consider refactoring soon     |
| 800-1000 lines | 🔴 Refactor before adding more   |
| > 1000 lines   | 🔴 **MUST refactor immediately** |

**When a file exceeds 500 lines:**

1. **Extract services** - Move business logic to `lib/*-service.ts`
2. **Split components** - Break into smaller, focused components
3. **Extract utilities** - Move helper functions to `lib/utils/`
4. **Separate concerns** - UI logic vs business logic vs data fetching

**Example Refactoring:**

```typescript
// ❌ BAD - 800 line page component
// app/workflows/page.tsx (800 lines)
'use client';
export default function WorkflowsPage() {
  // 800 lines of mixed concerns
}

// ✅ GOOD - Separated concerns
// app/workflows/page.tsx (150 lines) - Page shell
// components/workflows/WorkflowList.tsx (200 lines) - Display
// components/workflows/WorkflowFilters.tsx (100 lines) - Filtering
// lib/workflow-service.ts (150 lines) - Business logic
// lib/utils/workflow-helpers.ts (100 lines) - Utilities
```

### 🧪 Testing Requirements

**CRITICAL: Co-locate tests with implementation**

```
✅ CORRECT Structure:
app/
  workflows/
    page.tsx                    # Component
    page.test.tsx              # Test file (same directory!)
components/
  workflows/
    WorkflowList.tsx
    WorkflowList.test.tsx      # Co-located test
lib/
  workflow-service.ts
  workflow-service.test.ts     # Co-located test
```

**Testing Standards:**

- [ ] **Every new feature must have tests**
- [ ] **Tests live next to the file they test**
- [ ] **Test files named** `*.test.ts` or `*.test.tsx`
- [ ] **Cover happy path and error cases**
- [ ] **Run tests before PR:** `npm run test:merge-safe`

**Example Test Structure:**

```typescript
// components/WorkflowList.test.tsx
import { render, screen } from '@testing-library/react';
import { WorkflowList } from './WorkflowList';

describe('WorkflowList', () => {
  it('renders workflows', () => {
    const workflows = [{ id: '1', title: 'Test' }];
    render(<WorkflowList workflows={workflows} />);
    expect(screen.getByText('Test')).toBeInTheDocument();
  });

  it('handles empty state', () => {
    render(<WorkflowList workflows={[]} />);
    expect(screen.getByText('No workflows')).toBeInTheDocument();
  });

  it('handles errors gracefully', () => {
    // Test error scenarios
  });
});
```

### TypeScript Scope

**IMPORTANT:** TypeScript validation is **scoped to active features only**:

```
Validated paths:
  ✅ /admin/marqet/**
  ✅ /integrations/playbooks/**
  ✅ /api/marqet/**
  ✅ /api/playbooks/**

Excluded paths (not validated):
  ⏭️ /agents/, /meetings/, /demo-accounts/, /content/
```

## 3. Testing Commands

### Recommended Testing Workflow:

```bash
# ✅ MERGE-SAFE (Recommended before PR)
npm run test:merge-safe    # Runs lint + build (ignores TS warnings)

# Individual commands:
npm run lint               # Code style check
npm run build              # Production build verification
npm run type-check         # Show TS warnings (non-blocking)

# Run unit tests:
npm test                   # Run all tests
npm test -- --watch        # Watch mode for development

# E2E tests (optional):
npx playwright test --headed     # Visual debugging
npx playwright test              # Headless
```

### When to Run Each:

- **During development**: `npm test -- --watch` (unit tests in watch mode)
- **Before committing**: `npm run lint` (quick style check)
- **Before creating PR**: `npm run test:merge-safe` (full validation)
- **Before deployment**: `npm run build` (production verification)

## 4. Security Essentials 🔒

### CRITICAL Security Rules:

**Environment Variables vs Secrets:**

```bash
# ✅ Safe for .env.local (local development only)
NEXT_PUBLIC_SUPABASE_URL=http://127.0.0.1:54321
NEXT_PUBLIC_APP_URL=http://localhost:3002

# ⚠️ NEVER commit these (use .env.local, add to .gitignore)
SUPABASE_SERVICE_ROLE_KEY=eyJhbGc...
MARQ_API_KEY=sk_live_...
OAUTH_CLIENT_SECRET=abc123...
DATABASE_PASSWORD=...
```

**Security Checklist:**

- [ ] **Never commit secrets** - Check `.env.local` is in `.gitignore`
- [ ] **Use environment variables** - Never hardcode API keys
- [ ] **Validate user input** - Always sanitize data from users/external APIs
- [ ] **Use service role keys carefully** - Only in API routes, never client-side
- [ ] **Encrypt sensitive data** - OAuth tokens, API keys in database

**OAuth Token Handling:**

```typescript
// ✅ CORRECT - Store encrypted in database
const encryptedToken = await encryptToken(oauthToken);
await supabase.from('oauth_credentials').insert({
  encrypted_token: encryptedToken,
});

// ❌ WRONG - Never log or expose tokens
console.log('Token:', oauthToken); // NEVER DO THIS
res.json({ token: oauthToken }); // NEVER DO THIS
```

**If secrets are exposed:**

1. **Remove from git history** - Use `git filter-branch` or BFG Repo-Cleaner
2. **Rotate immediately** - API keys, OAuth secrets, database passwords
3. **Notify team** - Alert immediately
4. **Review access logs** - Check if compromised credentials were used

## 5. Performance Optimization ⚡

### Database Query Optimization

**Avoid N+1 Queries:**

```typescript
// ❌ BAD - N+1 query problem (1 + N queries)
const workflows = await supabase.from('workflows').select('*');
for (const workflow of workflows.data) {
  const accounts = await supabase
    .from('accounts')
    .eq('workflow_id', workflow.id);
}

// ✅ GOOD - Single query with join
const { data } = await supabase.from('workflows').select(`
    *,
    accounts (*)
  `);
```

**Always Paginate:**

```typescript
// ✅ Limit large result sets
const { data } = await supabase
  .from('workflows')
  .select('*')
  .range(0, 49) // Limit to 50 results
  .order('created_at', { ascending: false });
```

### React Performance

**Use Memoization:**

```typescript
// ✅ Memoize expensive computations
const sortedWorkflows = useMemo(() => {
  return workflows.sort((a, b) => a.priority - b.priority);
}, [workflows]);

// ✅ Prevent unnecessary re-renders
const handleClick = useCallback(() => {
  setSelected(workflow.id);
}, [workflow.id]);
```

**Code Splitting:**

```typescript
// ✅ Lazy load large components
const WorkflowEditor = dynamic(() => import('./WorkflowEditor'), {
  loading: () => <Skeleton />,
  ssr: false
});
```

### Next.js 14 App Router

**Server vs Client Components:**

```typescript
// ✅ Server Component (default) - Data fetching
// app/workflows/page.tsx
export default async function WorkflowsPage() {
  const workflows = await getWorkflows(); // Direct DB query
  return <WorkflowList workflows={workflows} />;
}

// ✅ Client Component - Interactivity
// components/WorkflowList.tsx
'use client';
export function WorkflowList({ workflows }) {
  const [selected, setSelected] = useState(null);
  // Interactive UI
}
```

**Monitor Slow Queries:**

```typescript
// ✅ Log queries over 1 second
const start = Date.now();
const { data } = await supabase.from('workflows').select('*');
const duration = Date.now() - start;

if (duration > 1000) {
  console.warn(`[SLOW QUERY] Took ${duration}ms`);
}
```

## 6. Database Setup & Seeding

### Local Supabase Database:

**IMPORTANT:** Always start with a fresh database when beginning a new session.

```bash
# 1. Check if Supabase is running
npx supabase status

# 2. If not running, start Supabase:
npx supabase start

# 3. Reset database and apply all migrations + seed data:
npx supabase db reset

# This will:
# - Drop and recreate the database
# - Apply all migrations from supabase/migrations/
# - Load seed data from supabase/seeds/init.sql
```

### What Gets Seeded:

1. **Production Accounts** - 5 real customer accounts (marqet_accounts)
2. **Production Workflows** - 9 workflows with correct UUIDs (marqet_workflows)
3. **Auth Users** - Team member accounts
4. **Test Data** - Sample playbooks, AI usage logs, token requests
5. **Workflow OAuth Config** - OAuth app credentials for Real Estate Playbooks
6. **Playbook-Workflow Links** - All playbooks linked to real-estate-playbooks workflow

**Note:** After seeding, run `npm run db:seed:oauth` to set encrypted OAuth credentials.

### ⚠️ Keep Seed Data in Sync with Migrations

**When you create new migrations:**

1. **Update seed file** - Add sample data to `supabase/seeds/init.sql`
2. **Test locally** - Run `npx supabase db reset`
3. **Commit together** - Commit migration + seed data together

```bash
# Example workflow:
npx supabase migration new add_workflow_tags
# Edit migration file
# Edit supabase/seeds/init.sql - add sample data
npx supabase db reset
git add supabase/migrations/ supabase/seeds/
git commit -m "feat: add workflow tags with seed data"
```

## 7. Environment Verification (Optional)

```bash
# Check Supabase connection
npx supabase status

# Check dev server
lsof -i :3002

# Start dev server if needed
npm run dev
```

## 8. Session Checklist

Before you start coding:

- [ ] On appropriate git branch (feature/fix, not main)
- [ ] Read relevant documentation (CLAUDE.md, feature docs)
- [ ] Environment variables configured (.env.local)
- [ ] **Database reset and seeded:** `npx supabase db reset`
- [ ] Dev server running: `npm run dev`
- [ ] Tests configured for new features (co-located with files)
- [ ] Understand file size limits (<1000 lines)
- [ ] Security checklist reviewed (no secrets in code)

## 9. Quick Reference

### Key Files by Feature:

**Playbooks:**

- `/app/integrations/playbooks/` - Playbook UI
- `/app/api/playbooks/` - API routes
- `/lib/workflow-oauth-service.ts` - OAuth service (centralized credentials)
- `/lib/execution-processor.ts` - Execution logic
- `/lib/server-token-refresh.ts` - Token refresh logic
- `/lib/token-encryption.ts` - AES-256-GCM encryption for tokens

**Key DB Tables (OAuth Architecture):**

- `workflow_oauth_configs` - OAuth app credentials per workflow
- `oauth_credentials` - User tokens per account+workflow
- `playbook_configurations.workflow_id` - Links playbook to workflow for OAuth
- `playbook_configurations.oauth_credential_id` - Links playbook to credential

**Marqet Admin:**

- `/app/admin/marqet/` - Admin UI
- `/app/api/admin/` - Admin APIs
- `/lib/marq-api-service.ts` - Marq API client

### Critical Patterns:

```typescript
// ✅ Foreign key filtering
const { data: industry } = await supabase
  .from('industries')
  .select('id')
  .eq('slug', industrySlug)
  .single();

query.eq('industry_id', industry.id);

// ✅ Service layer usage
import { TemplateService } from '@/lib/template-service';
const templates = await TemplateService.getAll('real-estate');

// ✅ API response handling
const data = Array.isArray(response)
  ? response
  : response.content || response.items || [];
```

### Database Migrations:

```bash
# Location: supabase/migrations/
# Format: YYYYMMDDHHMMSS_description.sql

# Create new migration:
npx supabase migration new description_of_change
```

**Migration Guidelines:**

1. **Always use timestamp format** - `YYYYMMDDHHMMSS_description.sql`
2. **Lowercase with underscores** - `add_oauth_tokens`
3. **All migrations in** `supabase/migrations/` - NOT in `scripts/`
4. **Update seed data** when adding tables/columns
5. **Test locally** before committing: `npx supabase db reset`
6. **Commit migration + seed data together**

## 10. Additional Resources

### For Deep Dives:

- **Engineering Standards:** `docs/engineering-standards.md`
  - API design patterns
  - Error handling & debugging
  - Code review guidelines
  - Rollback procedures
  - Dependency management
  - Observability & monitoring

- **Architecture:** `docs/ARCHITECTURE.md`
- **API Reference:** `docs/MARQ-API.md`
- **Common Tasks:** `docs/COMMON-TASKS.md`
- **Feature Docs:** `docs/features/*/`

## Ready to Code!

You're all set. Remember:

1. **Files < 1000 lines** - Refactor when needed
2. **Tests next to code** - Co-locate all tests
3. **Security first** - Never commit secrets
4. **Performance matters** - Avoid N+1, paginate, memoize
5. **Build success = merge ready**

Happy coding! 🚀

---

**Need Help?**

- Run `/begin` to see this again
- Check `docs/README.md` for documentation overview
- Review `docs/engineering-standards.md` for comprehensive guidelines
- Read feature-specific docs in `docs/features/`
