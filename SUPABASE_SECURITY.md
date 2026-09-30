# KARMSTHAN Supabase Security Contract

This document defines what the Supabase project must enforce before KARMSTHAN is treated as production-ready.

## 1. Authentication

All company-workspace operations require an authenticated Supabase user.

The application identifies the user through `auth.uid()`.

## 2. Tenant boundary

The canonical membership path is:

```
authenticated user
    ↓
company_members.user_id
    ↓
company_members.company_id
    ↓
company-owned record.company_id
```

A user must not be able to read or mutate another company's records by changing IDs in browser requests.

## 3. Tables requiring RLS

Enable and verify RLS for:

- `companies`
- `company_members`
- `jobs`
- `candidates`
- `applications`

Policies should cover the operations actually allowed by the product: SELECT, INSERT, UPDATE and DELETE where applicable.

## 4. Expected relationships

### companies

A company is owned/created by an authenticated user and has membership records.

### company_members

Membership connects an authenticated user to a company and contains `member_role`.

The current application creates the first membership with:

```
member_role = 'owner'
```

The application also recognizes recruiter/employer product roles at the authentication layer. Confirm the exact membership-role vocabulary in the Supabase schema before creating policies for additional roles.

### jobs

Every job belongs to a company through `jobs.company_id`.

### candidates

Every candidate belongs to a company through `candidates.company_id`.

### applications

An application belongs to a job and candidate. Its company boundary should be derived from the job's company rather than trusting a client-provided company ID.

## 5. Authorization model

The minimum production rule is:

> A user can access company-owned data only when that user has a corresponding row in `company_members`.

For mutations, add role-specific restrictions only after confirming the intended product permissions.

Do not use the following as a security mechanism:

- JavaScript checks alone
- hidden buttons
- client-side company IDs
- browser local storage
- application-level filtering without database policies

Those are UX controls, not authorization.

## 6. Applications

The application workflow currently validates:

1. The selected job belongs to the current company.
2. The selected candidate belongs to the current company.
3. The application status is one of the supported pipeline states.
4. An existing candidate/job application is detected before insert.

The database should still enforce the final boundary. If the schema supports it, add a unique constraint for the candidate/job pair so concurrent requests cannot create duplicates.

## 7. Resume storage

The resume bucket should remain private.

Storage policies should ensure that a user can access a resume only when the associated candidate belongs to a company to which that user has membership.

Never make the resume bucket public merely to simplify downloads.

## 8. Validation checklist

Before merging the MVP into `main`, test with at least two separate company accounts:

- Company A cannot read Company B jobs.
- Company A cannot read Company B candidates.
- Company A cannot read Company B applications.
- Company A cannot update Company B applications.
- Company A cannot upload/read Company B resumes.
- A user without company membership cannot access company data.
- An authenticated user cannot bypass the boundary by changing a UUID in the browser.

Also verify that the policies do not create recursive membership-policy failures.

## 9. Important implementation note

The repository does not currently contain Supabase migrations or a database schema export. Therefore, this repository intentionally does **not** contain guessed RLS SQL.

The next database-security step is to export/inspect the actual Supabase schema and existing policies, then implement tested policies against the real columns, foreign keys and roles.
