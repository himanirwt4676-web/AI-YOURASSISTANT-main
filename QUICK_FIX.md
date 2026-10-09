# Fix the chatbot connection

The supplied website calls `/functions/v1/chat`, while the Supabase project in your screenshots has a function named `dynamic-function`. The website also sends the public project API key as its bearer token instead of the signed-in user's access token. Its original backend uses OpenAI, not the working Gemini setup from your dashboard.

This corrected project calls `dynamic-function`, forwards the logged-in user's token, sends the public key in the `apikey` header, and reads Gemini responses. Errors now show useful messages. Your existing conversations and database schema are unchanged.

## 1. Use the corrected frontend

Extract the ZIP. Keep your original project as a backup.

Copy your existing `.env` into the extracted project. It needs these values:

```dotenv
VITE_SUPABASE_URL=https://udniakecpbfbpnpuwuzz.supabase.co
VITE_SUPABASE_ANON_KEY=YOUR_PROJECT_PUBLISHABLE_OR_ANON_KEY
VITE_CHAT_FUNCTION=dynamic-function
```

Replace the public-key placeholder with the public key from the SAME Supabase project. Do not use a secret/service-role key. The existing variable name `VITE_SUPABASE_ANON_KEY` accepts a publishable key too. Gemini's key belongs in Supabase Secrets, not the frontend.

In a terminal opened in this extracted project:

```bash
npm ci
npm run dev
```

Open the Local URL printed by Vite. Log in to the application and send a message. A restart is required after editing `.env`.

For a hosted website, replace its source with this corrected source, set the same three Vite environment values in its hosting settings, and rebuild/redeploy the frontend. Editing a Supabase function does not update an already deployed website's JavaScript.

## 2. Update the Supabase backend

The dashboard test already proves that your existing `dynamic-function` can contact Gemini. The corrected frontend can use that existing response format.

To deploy the backend included in this ZIP as well:

1. Open Supabase > your project > Edge Functions > Functions > dynamic-function > Code.
2. Replace its code with `supabase/functions/dynamic-function/index.ts` from this project.
3. Click Deploy updates.
4. In Edge Functions > Secrets, confirm that `GEMINI_API_KEY` is saved.
5. Keep the function's JWT verification enabled. The supplied backend also verifies the user with Supabase Auth and checks conversation ownership.

The included backend uses `gemini-2.5-flash`, the model that succeeded in your dashboard test. You may override it with a server secret named `GEMINI_MODEL` if your Gemini project requires another available model.

After deploying the included authenticated backend, test by logging into the website and sending a message. A dashboard test using only a public/anon project key will receive 401: it must supply a signed-in user's access token on Authorization to test this authenticated function.

If you only have Supabase access, give this ZIP to the person who maintains the website. They must apply and redeploy the frontend changes too.

## If it still fails

The website now reports the failing status instead of the old generic apology:

- Function not found: verify `VITE_CHAT_FUNCTION=dynamic-function` and that the frontend URL points to this project.
- Session rejected: log out and log in again; check that the frontend and function use the same Supabase project.
- Gemini quota: check the Gemini project quota/billing.
- Gemini request failed: inspect Edge Functions > dynamic-function > Logs for the upstream details.
- No conversations/messages: confirm the SQL migrations were applied to this project. Do not recreate or clear existing data.

Send the new error message and the failed request's status/response. Never share API keys or Authorization headers.

## Verification performed on this corrected ZIP

Regression tests mock the Supabase Auth and Gemini network boundaries, and exercise the real frontend transport and backend handler. They cover token/header placement, the deployed function name, fragmented UTF-8 responses, missing secrets, invalid login tokens, other-user conversations, quota failures, empty replies and CORS.

This work does not deploy changes to your live Supabase project or website. A signed-in live website test is still needed after you apply the files.
