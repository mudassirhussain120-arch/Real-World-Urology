# Real World Urology: going live

This folder is the whole website. `index.html` is the site, `_headers` adds two small security headers,
and `.github/workflows/keepalive.yml` is the daily job that stops the free database pausing.

## Part A. Supabase settings (do these before your first sign-in test)

1. **Authentication > Sign In / Providers > Email**: make sure Email is switched on.
2. **Authentication > Email Templates**: open **Confirm sign up** and replace the body with the text below.
   Do exactly the same for **Magic Link**. Without `{{ .Token }}`, the email will contain a link, not a code.

   Subject: `Your Real World Urology sign-in code`

   ```html
   <h2>Your sign-in code</h2>
   <p>Enter this code on the Real World Urology website:</p>
   <p style="font-size:28px;letter-spacing:4px"><strong>{{ .Token }}</strong></p>
   <p>It expires in one hour. If you did not ask for it, you can ignore this email.</p>
   ```
3. **Authentication > URL Configuration**: once the site has an address (Part B), set **Site URL** to it
   and add it under **Redirect URLs**.
4. **Email sending limit**: Supabase's built-in sender allows only about 2 emails an hour for the whole
   project. That is fine for you testing alone. Google sign-in sends no email, so it avoids this limit. Before you share the site widely, also connect a proper
   email service (this needs a domain).

## Part B. Put the site online (GitHub + Cloudflare Pages)

1. Create a free GitHub account, then a new repository called `real-world-urology` (private is fine).
2. Add the files: **Add file > Upload files** for `index.html`, `_headers` and `README.md`.
   For the job file choose **Add file > Create new file**, type the name `.github/workflows/keepalive.yml`
   (typing the slashes creates the folders), and paste in its contents.
3. In Cloudflare: **Workers & Pages > Create > Pages > Connect to Git**, pick the repository.
   Framework preset **None**, build command **empty**, output directory **/** (or leave blank). Deploy.
   You get an address like `real-world-urology.pages.dev`.
4. Go back to Supabase and set that address in Part A, step 3.
5. Later, add your own domain under the Pages project's **Custom domains**.
   Every change you push to GitHub updates the live site within a minute or two.

## Part C. Keep the free database awake

In the GitHub repository: **Settings > Secrets and variables > Actions > New repository secret**. Add two:
- `SUPABASE_URL` = `https://kvatamyfmezaptlwjtqu.supabase.co`
- `SUPABASE_KEY` = your publishable key (the `sb_publishable_...` one, never the secret key)

Then open the **Actions** tab, choose **Keep Supabase awake**, and click **Run workflow** once. A green tick means it works.
(If you move Supabase to the Pro plan, you can delete this file.)

## Part D. First test on the live address

1. Open the site, answer a question or two, then click **Sign up free** and enter your email.
2. Enter the 6-digit code from the email. You should land on a Welcome page; fill it in or skip.
3. In Supabase **Table Editor**, open `attempts`: your answers should be there.
4. Open the site on your phone, sign in with the same email, and check **My progress** matches.
5. Make yourself a reviewer: SQL Editor, run (with your email):
   `insert into public.reviewers (user_id) select id from auth.users where email = 'you@example.com';`
   Then refresh the site: **Review reports** appears in your account menu.
6. Send a test "Report a problem" and check it appears on the reviewer page.

## Testing practice without waiting 24 hours

In `index.html` find `PRACTICE_DELAY_HOURS:24` and change it to `0` in a separate test deployment only.
Do not point a test deployment at your real database if you want clean statistics.

## Google sign-in

The Google button is already switched on in this `index.html`. Once the site has an address:

1. Supabase > Authentication > URL Configuration: set **Site URL** to the address and add it under **Redirect URLs**
   (add both `https://your-address/` and `https://your-address/**`).
2. While Google's consent screen is in **Testing**, only the test users you listed can sign in with Google.
   When you are ready for real trainees: Google Auth Platform > Audience > **Publish app**.
3. Never put the Google client secret anywhere except Supabase.
