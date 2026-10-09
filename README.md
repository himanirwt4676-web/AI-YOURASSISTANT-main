# AI YOUR ASSISTANT

A React, TypeScript and Vite chatbot with Supabase authentication, saved conversations, avatars and a Gemini backend.

**For the chatbot failure in the uploaded project, start with [QUICK_FIX.md](QUICK_FIX.md).** It explains the confirmed function-name and authentication problems and how to apply this corrected project.

## Features

- Email/password authentication through Supabase Auth.
- Create, rename, search and delete conversations.
- Gemini replies delivered in the existing SSE response format.
- Saved message history with database Row Level Security.
- Profile and avatar management.
- Actionable errors for failed chatbot requests.

## Run locally

Copy `.env.example` to `.env` and fill in your existing project's public configuration. Keep secrets out of the frontend.

```bash
npm ci
npm run dev
```

Open the Local URL printed in the terminal, then sign in.

## Configuration

| Variable | Location | Purpose |
| --- | --- | --- |
| `VITE_SUPABASE_URL` | Frontend `.env` / hosting settings | Supabase project URL |
| `VITE_SUPABASE_ANON_KEY` | Frontend `.env` / hosting settings | Project publishable or legacy anon key |
| `VITE_CHAT_FUNCTION` | Frontend `.env` / hosting settings | Function name; defaults to `dynamic-function` |
| `GEMINI_API_KEY` | Supabase Edge Function Secrets | Server-side Gemini API key |
| `GEMINI_MODEL` | Supabase Edge Function Secrets, optional | Model override; defaults to the tested `gemini-2.5-flash` |

The frontend sends the user's session access token in `Authorization` and the public project key in `apikey`. The included Edge Function verifies the user with Supabase Auth and checks conversation ownership before contacting Gemini. It does not use a service-role key to read conversations.

Restart Vite after editing `.env`. Hosted builds require setting the frontend variables before building and redeploying.

## Supabase setup

Use your existing project. If creating a fresh project, apply the two SQL files in `supabase/migrations/` in filename order through the SQL Editor. These configure profiles, conversations, messages, owner policies, profile triggers and avatar storage. Migrations and functions are not automatically deployed simply by downloading this ZIP.

For dashboard deployment, open Edge Functions > `dynamic-function` > Code, paste `supabase/functions/dynamic-function/index.ts`, and click Deploy updates. Add `GEMINI_API_KEY` under Edge Function Secrets. Keep JWT verification enabled.

For CLI deployment from this project, with the Supabase CLI installed and authenticated:

```bash
supabase link --project-ref YOUR_PROJECT_REF
supabase functions deploy dynamic-function
```

If deploying the same handler under a different function name, set `VITE_CHAT_FUNCTION` to that exact name and rebuild the frontend.

## Relevant files

| File | Responsibility |
| --- | --- |
| `src/components/ChatView.tsx` | Messages, sending and display |
| `src/lib/chat.ts` | Authenticated chatbot request, errors and SSE parsing |
| `src/lib/supabase.ts` | Supabase client and function-name configuration |
| `supabase/functions/dynamic-function/index.ts` | Authenticated Gemini backend |
| `supabase/config.toml` | JWT verification configuration |
| `tests/chat.test.mjs` | Regression tests with mocked remote services |

## Checks

Use Node.js 20 or later for the test runner.

```bash
npm test
npm run typecheck
npm run lint
npm run build
```

Tests verify local logic with mocked network boundaries. They do not prove your deployed website's environment variables or remote project configuration; verify by signing into the live app after deployment.

## Hosting

Use `npm run build` and publish `dist` on your chosen host. Set the frontend environment variables there before building. Apply the updated source and rebuild: changing the backend alone does not replace the website's old request code.
