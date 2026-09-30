# Supabase Migration

Add or modify tables/columns, update TypeScript types, deploy changes.

## Projects using Supabase

| Project | Path |
|---------|------|
| cuefade | `C:\Work\git\cuefade` |
| garden | `C:\Work\git\garden` |

---

## 1. Write the migration SQL

Via Supabase MCP (preferred — no context switching):
> "Add a column `is_pro boolean default false` to the profiles table"

Or via Supabase dashboard → SQL Editor → New query:

```sql
-- example: add column
ALTER TABLE profiles ADD COLUMN IF NOT EXISTS is_pro boolean DEFAULT false;

-- example: create table
CREATE TABLE IF NOT EXISTS gardens (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id uuid REFERENCES auth.users(id) ON DELETE CASCADE,
  state jsonb NOT NULL DEFAULT '{}',
  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now()
);

-- RLS
ALTER TABLE gardens ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Users own their gardens"
  ON gardens FOR ALL
  USING (auth.uid() = user_id);
```

---

## 2. Update TypeScript types

### Option A — Supabase CLI (recommended)

```bash
npx supabase gen types typescript --project-id <project-id> --schema public > src/app/core/types/database.types.ts
```

Get `project-id` from Supabase dashboard URL: `supabase.com/dashboard/project/<id>`.

### Option B — Manual

Edit `src/app/core/types/database.types.ts` — update the `Tables` interface to match the new schema.

---

## 3. Update service code

If new columns added, update the relevant service's `.select()` call and the TypeScript interface it maps to.

Example (cuefade `auth.service.ts`):
```ts
// add new field to select
.select('id, email, is_pro, stripe_customer_id, new_column')

// update UserProfile interface
export interface UserProfile {
  id: string;
  email: string;
  isPro: boolean;
  newField: string;  // add
}

// update mapping
this._profile.set({
  ...
  newField: data.new_column,
});
```

---

## 4. Test locally

```bash
npm run build   # TypeScript must compile
npm test        # service specs must pass
```

---

## 5. Deploy

Supabase migrations run immediately in the dashboard SQL Editor — no separate deploy step.

For Edge Functions:
```bash
npx supabase functions deploy <function-name>
```

---

## 6. Commit

```bash
git add src/app/core/types/database.types.ts src/app/core/services/<changed>.service.ts
git commit -m "feat: <describe schema change>"
git push
```

---

## RLS policy patterns

```sql
-- Owner-only read/write
CREATE POLICY "owner" ON <table> FOR ALL USING (auth.uid() = user_id);

-- Authenticated read
CREATE POLICY "authenticated read" ON <table> FOR SELECT USING (auth.role() = 'authenticated');

-- Public read
CREATE POLICY "public read" ON <table> FOR SELECT USING (true);
```
