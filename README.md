# Fetch.ai LinkedIn Agent

Posts about **Fetch.ai** every day at **6:00 PM IST**.

- **uAgents** + **Agentverse** to host the agent
- **Chat Protocol** so you can talk to it on Agentverse / ASI:One
- **ASI:One** for post text and image
- **LinkedIn** to publish

---

## 1. Fill `.env`

Copy `.env.example` to `.env` if you do not already have one, then add:

```env
ASI1_API_KEY=your_asi_one_key
ASI1_BASE_URL=https://api.asi1.ai/v1

LINKEDIN_ACCESS_TOKEN=
LINKEDIN_AUTHOR_URN=

POST_HOUR=18
TIMEZONE_OFFSET=5.5

LINKEDIN_CLIENT_ID=your_linkedin_app_id
LINKEDIN_CLIENT_SECRET=your_linkedin_app_secret
LINKEDIN_REDIRECT_URI=http://localhost:8000/callback
```

Get an ASI:One key at [asi1.ai/developer](https://asi1.ai/developer).

On **Agentverse**, hosted agents already inject `ASI1_API_KEY` and `ASI1_BASE_URL`. You still need the LinkedIn keys in the agent `.env` / Secrets tab.

---

## 2. LinkedIn token (one-time)

1. Create an app at [linkedin.com/developers](https://www.linkedin.com/developers/apps)
2. Enable **Sign In with LinkedIn using OpenID Connect** and **Share on LinkedIn**
3. Add redirect URL `http://localhost:8000/callback`
4. Put Client ID and Secret in `.env`
5. Run:

```bash
pip install requests python-dotenv
python linkedin_setup.py
```

This writes `LINKEDIN_ACCESS_TOKEN` and `LINKEDIN_AUTHOR_URN` into `.env`.

The token lasts about **60 days**. Re-run the script when posting starts failing.

---

## 3. Host on Agentverse

1. Go to [agentverse.ai](https://agentverse.ai) → **Launch an Agent**
2. Open **Build** and paste `agent.py`
3. Open the agent **`.env` / Secrets** tab and paste the same keys from your local `.env`
4. Click **Run**

The Chat Protocol is already enabled in `agent.py` (`publish_manifest=True`). After it is running, use **Chat with Agent**.

---

## 4. Chat commands

- `post now` — write, generate image, publish now
- `preview` — write a post without publishing
- `status` — last post date and next 6pm slot

---

## 5. Agentverse profile

- **Name:** Fetch.ai LinkedIn Poster
- **Keywords:** fetch.ai, linkedin, uagents, agentverse, asi:one, daily post
- **README:** daily Fetch.ai LinkedIn poster. Uses ASI:One for text and images. Chat Protocol enabled.
