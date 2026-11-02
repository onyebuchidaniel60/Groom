# Groom — Environment Configuration

Use separate development/Preview and Production configurations. Never copy Production credentials into Preview. Values below are names and purpose only; put actual secrets in the hosting provider's secret manager or local ignored `.env.local`.

| Variable | Required | Server-only? | Source / notes |
|---|---:|---:|---|
| `NEXT_PUBLIC_APP_URL` | Yes | No | Exact canonical deployment host; set separately for Preview and Production. |
| `NEXT_PUBLIC_SUPABASE_URL` | Yes | No | Supabase project URL. |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Yes | No | Supabase publishable/anon key; RLS is mandatory. |
| `SUPABASE_SERVICE_ROLE_KEY` | Yes | Yes | Privileged server-only key; never bundle. |
| `SUPABASE_INPUT_BUCKET` | Yes | Yes | `respondent-inputs` private bucket. |
| `SUPABASE_RENDER_BUCKET` | Yes | Yes | `respondent-renders` private bucket. |
| `PARTICIPANT_TOKEN_HMAC_SECRET` | Yes | Yes | 32+ random bytes for participant token HMAC. |
| `RATE_LIMIT_HMAC_SECRET` | Yes | Yes | 32+ random bytes for hashed rate-limit keys; rotate using retention-aware migration. |
| `YOUCAM_API_BASE_URL` | Yes | Yes | Verified official host for the API account/region. |
| `YOUCAM_API_KEY` | Yes | Yes | Perfect Corp Bearer key. |
| `YOUCAM_UNIT_BUDGET` | Yes | Yes | Internal budget, not provider-reported balance. Default 700 only if 1,000-unit credit is confirmed. |
| `YOUCAM_HAIRSTYLE_PRESET_UNITS` | Yes | Yes | Current documented default 2; verify on account. |
| `YOUCAM_BEARD_UNITS` | Yes | Yes | Current documented default 2; verify on account. |
| `INNGEST_EVENT_KEY` | Yes | Yes | Inngest event key. |
| `INNGEST_SIGNING_KEY` | Yes | Yes | Inngest signing key. |
| `GROOM_NEW_SESSIONS_ENABLED` | Yes | Yes | Kill switch for new participant sessions. |
| `GROOM_RENDER_ENABLED` | Yes | Yes | Kill switch for new AI tasks. |
| `PREVIEW_MODE` | Yes | Yes | True in Preview; enables safe test guardrails, never bypass auth/RLS. |
| `LOG_LEVEL` | No | Yes | Default `info`; never log PII/secrets. |
| `GITHUB_REPOSITORY_URL` | No | Public-safe if public | Only actual approved public repository URL; unset for private/not-ready repo. |

## `.env.example` policy

The implementation must commit `.env.example` with variable names and obviously fake placeholder values such as `replace-me-locally`. It must never contain a working secret, key, real token, signed URL or production URL unless intentionally public. `.env.local` and all secret files must be gitignored.

## Startup validation

Parse all env values with Zod at server startup. Fail fast when mandatory server-only variables are missing. Validate `NEXT_PUBLIC_APP_URL` host/environment to prevent Preview links from using Production origins and vice versa. Never print actual env values in an error.
