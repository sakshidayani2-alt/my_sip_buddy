# My SIP Buddy: Netlify + Claude API

A public link that mentors can open without signing in to anything. The page is static. Typed questions go to SIP Buddy's agent: Claude, with a system prompt and six tools, called through one Netlify Function that holds your API key, so the key never reaches the browser. The tools run in the page on its own NSE data and maths. Buttons don't use AI at all.

```
public/index.html                          the page: start page, "How far into your SIP?" step, chat, term list, Without AI tools, Quality check (mentors)
netlify/functions/companion/companion.mjs  /api/companion: one agent turn on Claude per call, plus the rule check on every reply
netlify/functions/companion/core.mjs       the agent's system prompt, its tool definitions, the rule check and the personal-details scrubber
netlify/functions/market/market.mjs        /api/market: latest index levels from NSE Indices (no AI, no key needed)
netlify.toml                               tells Netlify where things are (no build step)
```

## 1. Make a Claude API key (about 3 minutes)

It uses the API credit already in your Claude Console account.

1. Sign in to the Claude Console: https://platform.claude.com
2. Recommended: make a workspace for this site, so its spend is separate and capped. Go to **Settings → Workspaces → Create workspace** and name it `SIP Buddy`. Open it, click **Spend limits** and set a monthly limit (for example $5).
3. Go to **Settings → API keys** (https://platform.claude.com/settings/keys) and click **Create key**. Name it `sip-buddy-netlify` and pick the `SIP Buddy` workspace. A key made inside a workspace needs no extra setting.
4. Copy the key (it starts with `sk-ant-`). The Console shows it only once.
5. Keep it private. Don't paste it into chats, files or GitHub. It only goes into Netlify (step 3).

## 2. Put this folder on GitHub (no terminal needed)

1. On GitHub, create a new repository. Don't use "zerodha" in the name.
2. On the empty repository page, choose **uploading an existing file**.
3. Drag in everything inside this folder (`netlify.toml`, `README.md`, `public`, `netlify`), then commit.

Already deployed? Upload just the changed `netlify` and `public` folders with **Add file → Upload files**, then commit.

## 3. Add the key in Netlify and redeploy

1. In Netlify, open your site, then **Site configuration → Environment variables → Add a variable → Add a single variable**.
   - Key: `ANTHROPIC_API_KEY`
   - Value: your key from step 1
   - Scopes: leave **All scopes** (or at least **Functions**). Values: **Same value for all deploy contexts**.
2. Click **Create variable**.
3. Redeploy so the function picks up the key: **Deploys → Trigger deploy → Deploy site**.

Check it: open `https://<your-site>.netlify.app/api/companion`. You should see `"configured":true,"provider":"anthropic","model":"claude-haiku-5-5"`.

You can leave `GEMINI_API_KEY` in Netlify. With `ANTHROPIC_API_KEY` set, the site uses Claude.

## 4. Run the quality check (switches AI replies on)

AI replies stay off until this version (agent-v2) passes the 96-case test set on this model. Until then, typed questions get the pre-written replies and the chat header says "Pre-written replies". Don't start tester sessions before this step.

1. Open `https://<your-site>.netlify.app/?view=mentor#quality` and click **Run the 96 cases**.
2. Keep the tab open. It takes 10 to 15 minutes: each answer takes 2 to 4 short requests, about 300 in all, well under a dollar of credit on Claude Haiku 5.5. If it hits a rate limit, it waits and retries.
3. Click **Copy results** and paste them into the chat with Claude.
4. Claude sends back a new `public/index.html` with the results built in. Replace that one file on GitHub. Netlify redeploys by itself, and AI replies switch on if the check passed.

## Optional settings (Netlify environment variables)

| Variable | Default | What it does |
| --- | --- | --- |
| `ANTHROPIC_MODEL` | `claude-haiku-5-5` | The Claude model. Changing it switches AI replies off until the quality check is run again on the new model. |
| `CLAUDE_THINKING` | off | `adaptive` lets the model think before each step: slower, and a reply may not fit the page's 30-second limit. |
| `CLAUDE_EFFORT` | `low` | `low`, `medium` or `high`. Lower is faster and cheaper. |
| `ANTHROPIC_WORKSPACE_ID` | none | Only for a key that isn't tied to a workspace (the API says so if it's needed). |
| `AI_PROVIDER` | `anthropic` when `ANTHROPIC_API_KEY` is set | The agent runs on Claude only. Setting `gemini` switches AI replies off (the older two-step prompts are kept in the code for reference). |
| `GEMINI_MODEL`, `GEMINI_THINKING_LEVEL` | `gemini-3.5-flash-lite`, `minimal` | Only used when `AI_PROVIDER` is `gemini`. |
| `MAX_REQUESTS_PER_10_MIN` | `600` | Rough cap per visitor, so one person can't use up the credit. The quality check needs about 300. |

## Good to know

- **How the agent works:** for each typed message, Claude reads the conversation, decides which tools it needs (facts on 47 topics, a fund-type comparison over any time span, a goal check, a check of your own SIP, failed-payment help, a tax estimate), the page runs them, and Claude writes the reply. The system prompt holds the rules: data and insight, never a pick; never tell people what to do; no predictions, promises, pressure or fund names; numbers only from tools.
- **Cost:** a typed message makes 2 to 4 short calls, a fraction of a cent on Claude Haiku 5.5. Buttons cost nothing.
- **Personal details:** Anthropic doesn't use what's sent through its API to train its models unless you opt in. The page still removes PAN, account numbers, phone numbers and email addresses before sending, the function removes them again, and the page asks people not to type them.
- **What it knows:** 47 topics, each answered from sourced facts on the page: starting out (demat account, KYC, minimum amount), planning (horizon, risk tolerance, compounding, step-up), every fund type with its own card (a flexi-cap question shows only flexi cap), equity vs debt funds, tax on gains with the old and new Income-tax Act sections and STT, and SEBI, SCORES and IEPF. A bare amount like "2000" after "how much do you put in each month?" is read as the monthly amount by the page itself.
- **Voice:** the mic button in the chat uses the browser's own speech-to-text (English or Hindi; in Chrome that's Google's speech service). The words fill the box for the person to check and send. Browsers without it, such as Firefox, show no mic. Nothing to set up in Netlify.
- **Questions outside SIPs and mutual funds:** the agent only marks them off-topic. The reply is a friendly fixed message from the server about what SIP Buddy is for; the AI writes nothing for it.
- **When a reply breaks a rule:** the rule check sends it back to the agent once, with the reason. If the rewrite still breaks a rule, the page shows its pre-written answer and says why.
- **When the AI is busy, slow (over 30 seconds) or out of credit:** chat replies fall back to the pre-written answers, with a note saying why. Nothing breaks.
- **No market headlines:** the page doesn't show news. Headlines put daily moves, and other people's fear or FOMO, in front of first-year investors, which works against the goal.
- **Buttons vs typing:** the starter buttons ask for details first and answer from the page's own maths and NSE index history, instantly and without AI. Typed questions go through the AI and its rule check.
- **Index history:** month-end total return values for the Nifty 50, Midcap 150, Smallcap 250 and Nifty 500 from April 2005 to October 2026 (NSE Indices) are built into the page, plus debt and liquid index returns to 30 September 2026 (NSE Indices fixed income dashboard). Refresh them each month for current figures.
- **Not investment advice:** the rule check runs on the server and blocks any reply that names a fund or fund house, steers toward a type of fund, tells someone what to do with their investments, promises returns, uses pressure, repeats an ID number, or uses a number the page didn't work out.

### Command-line alternative to steps 2 and 3

```
npm install -g netlify-cli
netlify login
netlify deploy --prod          # run inside this folder; creates the site on first run
netlify env:set ANTHROPIC_API_KEY "paste-your-key-here"
netlify deploy --prod          # redeploy so the function sees the key
```
