# KARMSTHAN Hiring Intelligence Platform

KARMSTHAN is an AI-assisted recruitment and hiring outcome management platform.

## Current MVP flow

Employer / Recruiter → Jobs → Candidates → Matching → Hiring Readiness → Applications → Pipeline

The V1 Hiring Intelligence engine is **rule-based and explainable**. It currently evaluates signals such as skills, experience, compensation and notice period. It is not presented as a trained machine-learning model.

## Supabase security requirement

KARMSTHAN uses Supabase as its application data layer. The browser performs company-scoped queries for usability, but **frontend filtering is not a security boundary**.

Before production deployment, Supabase Row Level Security (RLS) must enforce tenant isolation for:

- `companies`
- `company_members`
- `jobs`
- `candidates`
- `applications`

The intended authorization relationship is:

`auth.uid() → company_members.user_id → company_members.company_id`

Company-owned records should only be readable/writable when the authenticated user is a member of the record's `company_id`.

Application access should additionally be constrained through the application's job → company relationship.

Do not copy generic policies without checking the actual table definitions, foreign keys, and existing policies in the Supabase project.

See [SUPABASE_SECURITY.md](./SUPABASE_SECURITY.md) for the implementation contract.

## Resume storage

Resume files are expected to live in a **private** Supabase Storage bucket. Storage policies should follow the same company-membership boundary and must not rely on a client-supplied company ID alone.

## Development branches

- `main` — production/stable branch
- `mvp-development` — active MVP development

Changes should be validated on `mvp-development` before any merge into `main`.
