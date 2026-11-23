# jewelrygenerator
AI model for jewelry creation
# Beau Studio

A private design studio for Beau clients. They describe a piece in their own words ("big but soft",
"like my grandmother's but modern", or a Pinterest photo) and get:

- **An image** of the piece, made by Google's Nano Banana image model
- **The design in jeweler terms**, e.g. `18K YELLOW GOLD · ELONGATED OVAL · 4 CLAW PRONGS`
- **Words to know**, explaining the jewelry vocabulary they just "used" without knowing it
- **Beau's note**, an honest check on whether a real bench jeweler could make it
- **Before / after** for every edit, **try-on** on their own photo, **Save**, and **Bring this to Gabriela**
  (downloads the image and opens a pre-filled email to you)

Piece types each get their own set of rules, so designs don't drift from one type into another:
engagement ring · signet ring · cocktail ring · earrings · pendant & necklace · bracelet · bolo tie.

---

## How it works

```
Client message ─▶ 1. TRANSLATE  (Gemini text model)
                     vibes → jeweler spec, Beau's note, words to know, next-edit suggestions
                     also notices "that's actually earrings" and sends them to the right tab
               ─▶ 2. GENERATE   (Nano Banana image model)
                     house style + that piece type's rules + manufacturability rules
               ─▶ 3. QA CHECK   (Gemini text model looks at the image)
                     right piece? plain background? stones actually held?  → one retry if not
Try-on photo   ─▶ check it's a hand / ear / neck / wrist → place the piece on it
```

| File | What it is | Edit it when… |
|---|---|---|
| `lib/categories.js` | Piece types, starter designs, vibe chips, rules per type, house style, bench rules | you want a new piece type, different chips, or tighter rules |
| `lib/prompts.js` | The instructions for the translate, generate, QA and try-on steps | you want to change Beau's voice or how strict the checks are |
| `api/*.js` | The server code (keeps your API key secret) | rarely |
| `public/*` | The page clients see | you want to change the look |

---

## Go live in about 20 minutes

### 1. Get a Gemini API key and cap the spend first
1. Go to **aistudio.google.com**, sign in, then **Get API key → Create API key**.
2. Set up billing for the project (image models need the paid tier).
3. **Before anything else**, open the **Spend** tab and set a **Monthly spend cap** (start with $10–$20).
   Once spend hits the cap, Google blocks further calls, so a leaked key can't run up a big bill.

### 2. Put the code on GitHub
Create a new **private** repository and upload this folder. `.env.local` is in `.gitignore`. Never commit your key.

### 3. Deploy on Vercel (the free plan is fine)
1. Go to **vercel.com → Add New → Project**, then import the repository.
2. Under **Environment Variables**, add:

| Name | Value |
|---|---|
| `GEMINI_API_KEY` | your key from step 1 |
| `ACCESS_CODES` | comma-separated codes, e.g. `BEAUEVENT,CLIENT-SOFIA` |
| `CONTACT_EMAIL` | where "Bring this to Gabriela" emails go |

3. Click **Deploy**. Your studio goes live at `your-project.vercel.app`. You can add a subdomain like `studio.beau…` under **Settings → Domains**.

Optional variables:
- `GEMINI_IMAGE_MODEL`: default `gemini-3.1-flash-image`. Try `gemini-3-pro-image` for sharper detail (costs more).
- `GEMINI_TEXT_MODEL`: default `gemini-3.8-flash`.
- `IMAGE_SIZE`: default `1K`. `2K` gives crisper stones at higher cost.

Check current per-image prices on Google's Gemini API pricing page before your event.

### 4. Put it on dearbeau.com (Framer)
1. In Framer, add a new page, e.g. **/studio**. Keep it out of the site nav.
2. **Insert → Embed**, choose the **URL** option, and paste your Vercel link (`https://your-project.vercel.app`).
3. Set the embed's width to **Fill** and its height to about **900px** (or 100vh). On the phone breakpoint, use about **1400px** height, because the chat stacks under the design.
4. In the page's SEO settings, hide it from search engines.
5. Publish. Clients go to **dearbeau.com/studio** and see the Beau Studio password screen inside your site.

When it's inside the embed, the studio hides its own "Beau Studio" header, since your Framer nav is already there.
It also refuses to load inside any site other than dearbeau.com and Framer (set in `vercel.json`).
If you later add a custom domain to Framer, add it to the `frame-ancestors` line in `vercel.json`.

**About the password:** the password screen inside the studio is the real lock, because it's checked on the server.
Most Framer password plugins only hide the page in the browser, and anyone who views the page source
can see the embed link. That doesn't matter here: the link alone gets them to the password screen, nothing more.
You can still add a Framer page-password plugin as a branded outer door if you like the look.

### Access codes (the passwords)
- Give each event its own code (`HOLIDAY-EVENT`) and each serious client their own (`CLIENT-SOFIA`).
- To shut a code off, remove it from `ACCESS_CODES` in Vercel and redeploy.
- If `ACCESS_CODES` is empty, the studio is **open to anyone**, so always set it.

---

## Try it on your laptop

```bash
npm run mock      # no key, no cost — placeholder images, every button works
# or, with a real key:
cp .env.example .env.local   # fill in GEMINI_API_KEY
npm run dev
```
Then open http://localhost:3000. Requires Node 18+.

---

## Ideas for v2

- **"Customize this piece" from your store**: open the studio with one of your real pieces pre-loaded as the starting image.
- **Your own rules**: add the things you always tell clients to `BENCH_RULES` (e.g. minimum prong count, your preferred setting heights).
- **Save designs for you**: store each design plus the conversation so you can see what clients made before their appointment.
- **Per-code limits**: cap how many images each access code can make (needs a small database such as Upstash or Vercel KV).
- **Ring sizer and booking**: link "Bring this to Gabriela" to your Calendly instead of email.

## Safety checklist before sharing the link
- [ ] Spend cap set in AI Studio
- [ ] `ACCESS_CODES` set
- [ ] API key only in Vercel's environment variables. Never in the code, in GitHub, or in a message.
- [ ] Ran one design, one edit and one try-on on the live site
