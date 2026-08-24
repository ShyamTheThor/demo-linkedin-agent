# LinkedIn Buddy

Workshop guide: build a Fetch.ai agent that writes a LinkedIn post with **ASI:One**, generates an image with **ASI:One**, and publishes it to LinkedIn every day at **6:00 PM IST**.

You can chat with it on Agentverse / ASI:One using the **Chat Protocol**.

Repo: [github.com/ShyamRV/demo-linkedin-agent](https://github.com/ShyamRV/demo-linkedin-agent)

---

## What you will build

```text
You (chat)  -->  LinkedIn Buddy (uAgent)
                      |
                      +--> ASI:One writes the post
                      +--> ASI:One generates an image
                      +--> LinkedIn API publishes it
                      +--> every day at 6:00 PM it posts by itself
```

**Stack**

| Piece | What it does |
| --- | --- |
| **uAgents** | Agent framework (Fetch.ai) |
| **Agentverse** | Host / discover / chat with the agent |
| **Chat Protocol** | Lets ASI:One and Agentverse talk to the agent |
| **ASI:One** (`asi1.ai`) | Writes the post + generates the image |
| **LinkedIn UGC API** | Publishes the post on your profile |

---

## Accounts you need (create these first)

Do these **before** writing any code.

### 1. Fetch.ai / Agentverse

1. Open [agentverse.ai](https://agentverse.ai)
2. Sign up / log in
3. Keep this tab open. You will host or connect the agent here later.

### 2. ASI:One API key

1. Open [asi1.ai](https://asi1.ai) and sign in
2. Go to [asi1.ai/developer](https://asi1.ai/developer) (or Dashboard → API keys)
3. Create an API key
4. Copy it. It looks like `sk_...`
5. You will paste it into `.env` as `ASI1_API_KEY`

Hosted agents on Agentverse already get `ASI1_API_KEY` injected. You still need this key for **local** runs.

### 3. LinkedIn Developer App

1. Open [linkedin.com/developers/apps](https://www.linkedin.com/developers/apps)
2. Click **Create app**
3. Fill name, LinkedIn page, and logo (LinkedIn requires a page + logo)
4. Open the app → **Products** and request / enable:
   - **Sign In with LinkedIn using OpenID Connect**
   - **Share on LinkedIn**
5. Open **Auth**
6. Copy **Client ID** and **Client Secret**
7. Under **Authorized redirect URLs for your app**, add **exactly**:

```text
http://localhost:8000/callback
```

Must match character-for-character:

- `http` not `https`
- `localhost` not `127.0.0.1`
- no trailing slash
- port `8000`

If this is wrong you will see: `The redirect_uri does not match the registered value`.

---

## Files in this project

| File | Purpose |
| --- | --- |
| `agent.py` | The whole agent. Paste this into Agentverse **Build**. |
| `.env` | Your secrets. **Do not commit or share this.** |
| `.env.example` | Empty template of the same keys |
| `linkedin_setup.py` | One-time script that gets the LinkedIn token + author URN |
| `requirements.txt` | Python packages |
| `README.md` | This workshop guide |

---

## Step 1 — Get the code

```powershell
git clone https://github.com/ShyamRV/demo-linkedin-agent.git
cd demo-linkedin-agent
```

Or download the ZIP from GitHub and unzip it.

---

## Step 2 — Python 3.10+ (required)

This agent needs **Python 3.10 or newer**. Python 3.8 cannot install current `uagents` / `openai`.

Check:

```powershell
python --version
```

If you see `3.8` or nothing, install Python 3.12 from [python.org/downloads](https://www.python.org/downloads/).

On this workshop machine we used:

```powershell
& "$env:LOCALAPPDATA\Programs\Python\Python312\python.exe" --version
```

You should see `Python 3.12.x`.

Install packages:

```powershell
& "$env:LOCALAPPDATA\Programs\Python\Python312\python.exe" -m pip install -r requirements.txt
```

If `python` is already 3.10+, this is enough:

```powershell
python -m pip install -r requirements.txt
```

Packages installed: `uagents`, `openai`, `requests`, `python-dotenv`.

---

## Step 3 — Create `.env`

1. Copy `.env.example` to `.env`
2. Open `.env` and fill the values below

```env
# ASI:One  — from https://asi1.ai/developer
ASI1_API_KEY=sk_your_key_here
ASI1_BASE_URL=https://api.asi1.ai/v1

# LinkedIn  — leave token/URN empty until Step 4
LINKEDIN_ACCESS_TOKEN=
LINKEDIN_AUTHOR_URN=

# Schedule (18 = 6pm, 5.5 = India IST)
POST_HOUR=18
TIMEZONE_OFFSET=5.5

# Agent identity
AGENT_NAME=LinkedIn Buddy
AGENT_HANDLE=linkedin-buddy
AGENT_SEED=linkedin-fetchai-poster-seed
AGENT_PORT=8001

# LinkedIn app  — from LinkedIn Developers → Auth
LINKEDIN_CLIENT_ID=your_client_id
LINKEDIN_CLIENT_SECRET=your_client_secret
LINKEDIN_REDIRECT_URI=http://localhost:8000/callback
```

**Handle rule:** use `linkedin-buddy` (hyphen). Do **not** use `linkedin buddy` (space). Agentverse handles are max 20 characters and cannot have spaces.

Do not put real secrets in GitHub. `.env` is gitignored.

---

## Step 4 — Get LinkedIn access token and author URN

This is a one-time login. The script writes two values into `.env`:

- `LINKEDIN_ACCESS_TOKEN` — lets the agent post as you
- `LINKEDIN_AUTHOR_URN` — `urn:li:person:xxxxxxxx` (you, the author)

### 4.1 Run the setup script (PowerShell)

```powershell
& "$env:LOCALAPPDATA\Programs\Python\Python312\python.exe" .\linkedin_setup.py
```

If `python` is 3.10+:

```powershell
python .\linkedin_setup.py
```

PowerShell notes:

- Do **not** use `"%LOCALAPPDATA%\..."` — that is CMD syntax
- Do **not** run `linkedin_setup.py` alone — use `.\linkedin_setup.py` or `python .\linkedin_setup.py`

### 4.2 Log in to LinkedIn

The script prints a URL. Open it, log in, click **Allow**.

### 4.3 Copy the `code`

LinkedIn redirects to a page like:

```text
http://localhost:8000/callback?code=AQS...
```

The page **fails to load**. That is expected. You do not need a server on port 8000.

Copy **only** the value after `code=` (stop before `&` if there is one).

Paste it into the terminal where it says `Paste the code here:` and press Enter.

The code expires in a few minutes. If it fails, run the script and log in again.

### 4.4 Confirm `.env`

You should now see filled values:

```env
LINKEDIN_ACCESS_TOKEN=AQV...
LINKEDIN_AUTHOR_URN=urn:li:person:xxxxxxxx
```

The token lasts about **60 days**. When posting starts returning 401, run `linkedin_setup.py` again.

To post as a **company page** instead of your profile, use `urn:li:organization:YOUR_ID` and the `w_organization_social` scope.

---

## Step 5 — Run the agent locally

```powershell
& "$env:LOCALAPPDATA\Programs\Python\Python312\python.exe" .\agent.py
```

Good logs look like this:

```text
INFO: [LinkedIn Buddy]: Starting agent with address: agent1q...
INFO: [LinkedIn Buddy]: Agent inspector available at https://agentverse.ai/inspect/?uri=...
INFO: [LinkedIn Buddy]: Starting server on http://0.0.0.0:8001
INFO: [LinkedIn Buddy]: Manifest published successfully: AgentChatProtocol
INFO: [uagents.registration]: Registration on Almanac API successful
```

### Expected warnings (safe to ignore)

| Warning | Meaning |
| --- | --- |
| `I do not have enough funds to register on Almanac contract` | No FET in the wallet. API registration already worked. |
| `LinkedIn secrets are empty` | Step 4 is not done. Fill token + URN, then restart. |
| `ASI1_API_KEY is empty` | Add the ASI:One key to `.env`, then restart. |

Leave this terminal running. Do not close it.

---

## Step 6 — Connect the mailbox (required for chat)

A local agent cannot receive Agentverse / ASI:One chat until the mailbox is connected.

1. Copy the **Agent inspector** URL from the terminal
2. Open it while the agent is running
3. Click **Connect**
4. Choose **Mailbox**
5. Confirm the log shows mailbox connected

Now you can use **Chat with Agent** on Agentverse.

If you skip this, chat will not reach the local agent.

---

## Step 7 — Talk to the agent

Use **Chat with Agent** on Agentverse, or ASI:One with **`@linkedin-buddy`**.

| You type | What happens |
| --- | --- |
| `help` | Shows commands |
| `status` | Last post + next 6pm slot |
| `preview` | Draft today's Fetch.ai post (does **not** publish) |
| `preview about uAgents` | Draft about that topic |
| `post now` | Write image + publish today's Fetch.ai post |
| `post about <topic or profile>` | Write image + publish that topic |

**Always test with `preview` first.** `post now` and `post about` publish for real.

Example:

```text
preview about Fetch.ai Agentverse and ASI:One
```

Then:

```text
post about Fetch.ai Agentverse and ASI:One
```

You can also paste a LinkedIn profile after `post about` and it will write a post about that person.

---

## Step 8 — Host on Agentverse (workshop deploy)

There are two ways to put an agent on Agentverse. Use **Hosted Agent**. Do **not** use External Integration unless you have a public HTTPS URL.

### Do this

1. Go to [agentverse.ai](https://agentverse.ai)
2. Click **Launch an Agent**
3. Choose **Generate Agent** or a **blank Hosted Agent**
4. **Do not** choose External Integration (that asks for an Endpoint URL — you do not have one)
5. Open **Build**
6. Paste the full contents of `agent.py`
7. Open **`.env` / Secrets**
8. Paste the same keys from your local `.env` (at least):
   - `LINKEDIN_ACCESS_TOKEN`
   - `LINKEDIN_AUTHOR_URN`
   - `POST_HOUR=18`
   - `TIMEZONE_OFFSET=5.5`
9. Click **Run**

Hosted agents already receive `ASI1_API_KEY` and `ASI1_BASE_URL`. Extra `Agent(name=..., port=..., mailbox=True)` arguments are ignored on hosted agents. That is normal.

### What the Endpoint URL field is (skip this)

If you see **Agent Endpoint URL**, you are on the **external agent** path.

That field is a **public HTTPS address** where Agentverse can send messages to an agent running on your own server, for example:

```text
https://your-server.com:8000/submit
```

`localhost` will not work there. For this workshop, go back and create a **Hosted Agent** instead.

---

## Step 9 — Agentverse profile (so people can find it)

In the agent dashboard, set:

| Field | Value |
| --- | --- |
| **Name** | LinkedIn Buddy |
| **Handle** | `@linkedin-buddy` (no space) |
| **Keywords** | fetch.ai, linkedin, uagents, agentverse, asi:one, daily post, linkedin-buddy |
| **Description** | Posts about Fetch.ai on LinkedIn every day at 6pm. Uses ASI:One for text and images. |
| **README** | LinkedIn Buddy writes and publishes a daily Fetch.ai LinkedIn post. Chat Protocol enabled. Say `preview`, `post now`, or `post about <topic>`. |

After it is running, ASI:One can find it as **`@linkedin-buddy`**.

---

## How the agent works (for the workshop talk)

1. **`agent.py` is the whole agent.** One file, so it is easy to paste into Agentverse.
2. **`.env` holds secrets.** `load_dotenv()` reads them locally. On Agentverse, use the Secrets tab.
3. **Chat Protocol** is attached with `Protocol(spec=chat_protocol_spec)` and `agent.include(protocol, publish_manifest=True)`. That is what makes it show up as `AgentChatProtocol`.
4. **ASI:One chat** (`https://api.asi1.ai/v1/chat/completions`, model `asi1`) writes the LinkedIn text.
5. **ASI:One image** (`https://api.asi1.ai/v1/image/generate`) creates the picture.
6. **LinkedIn** uploads the image, then creates a UGC post.
7. **`@agent.on_interval(period=60)`** checks every minute. At 6:00 PM (`POST_HOUR=18`) it posts once per day and stores the date in agent storage.
8. **Chat** can also trigger `preview` / `post now` / `post about ...` at any time.

Daily topics rotate (Fetch.ai, uAgents, Agentverse, ASI:One, Chat Protocol, use cases, decentralized AI).

---

## Restart after changing `.env`

The running agent does not reload `.env` by itself. Stop it (Ctrl+C) and start `agent.py` again.

---

## Troubleshooting

| Problem | Fix |
| --- | --- |
| `Unexpected token 'linkedin_setup.py'` | You used CMD `%LOCALAPPDATA%` in PowerShell. Use `& "$env:LOCALAPPDATA\Programs\Python\Python312\python.exe" .\linkedin_setup.py` |
| `linkedin_setup.py is not recognized` | Run `python .\linkedin_setup.py` (note the `.\`) |
| `redirect_uri does not match` | Add `http://localhost:8000/callback` exactly in the LinkedIn app Auth settings, then try again |
| Localhost page will not load after LinkedIn login | Expected. Copy `code=` from the address bar |
| Code expired / invalid | Run the setup script and log in again immediately |
| `No matching distribution for jiter` / openai install fails | Python is too old. Use 3.10+ |
| Chat does not respond | Agent must be running. Local agents also need **Connect → Mailbox** |
| Agent replies with help instead of posting | Say `post about ...` or `post now`. Handle is `@linkedin-buddy` not `@linkedin buddy` |
| `LinkedIn secrets are empty` | Finish Step 4 and restart |
| Endpoint URL required | You created an External agent. Create a **Hosted Agent** instead |
| 401 from LinkedIn | Token expired (~60 days). Re-run `linkedin_setup.py` |
| Almanac contract funds warning | Safe to ignore for the workshop |

---

## Workshop checklist

- [ ] Agentverse account
- [ ] ASI:One API key in `.env`
- [ ] LinkedIn app with OpenID + Share on LinkedIn
- [ ] Redirect URL `http://localhost:8000/callback`
- [ ] Client ID + Secret in `.env`
- [ ] `linkedin_setup.py` wrote token + URN
- [ ] Python 3.10+ and `pip install -r requirements.txt`
- [ ] `agent.py` running locally
- [ ] Inspector → Connect → Mailbox
- [ ] `preview` works
- [ ] Hosted Agent on Agentverse with the same secrets
- [ ] Profile name **LinkedIn Buddy**, handle **`@linkedin-buddy`**

---

## Useful links

- Agentverse: [agentverse.ai](https://agentverse.ai)
- ASI:One keys: [asi1.ai/developer](https://asi1.ai/developer)
- Chat Protocol: [docs.agentverse.ai — Enable the Chat Protocol](https://docs.agentverse.ai/documentation/getting-started/enable-chat-protocol)
- LinkedIn apps: [linkedin.com/developers/apps](https://www.linkedin.com/developers/apps)
- This repo: [github.com/ShyamRV/demo-linkedin-agent](https://github.com/ShyamRV/demo-linkedin-agent)
