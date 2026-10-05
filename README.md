# EpinetLens

EpinetLens is a chat-based epistemic network analyst. Paste in a document, post or article and it walks you through a step-by-step analysis of how credibility, trust, competence and legitimacy circulate among the people, claims and institutions in the text. The analysis follows the framework of *Epinets: The Epistemic Structure and Dynamics of Social Networks* (Moldoveanu & Baum, 2014).

## How it works

The app has two parts:

| File | Runs on | Job |
| --- | --- | --- |
| `index.html` | GitHub Pages (or any static host) | The whole user interface: chat, persistent context panel, reset button |
| `epinetlens-worker.js` | Cloudflare Workers | Holds the Anthropic API key and calls Claude on the page's behalf |

The browser never sees the API key. The page sends the conversation and any context to the Worker; the Worker adds the system prompt, model and reply-length cap, calls Anthropic's Messages API, and streams the reply back.

There is no database and no server-side memory. The conversation lives in the browser tab and is resent each turn, so closing or reloading the page starts a fresh conversation.

## Setup

### 1. Deploy the Worker

1. Sign in at [dash.cloudflare.com](https://dash.cloudflare.com) and open **Workers & Pages**.
2. Create a new Worker from the Hello World starter and name it (e.g. `epinetlens`).
3. Open the code editor, replace the placeholder with the contents of `epinetlens-worker.js`, and deploy.
4. In the Worker's **Settings**, under **Variables and Secrets**, add a variable of type **Secret** named `ANTHROPIC_API_KEY` with your Anthropic key as the value, and deploy.
5. Note the Worker's URL, which looks like `https://epinetlens.<your-subdomain>.workers.dev`.

To check it, open the Worker's URL in a browser. You should see `{"error":"Only POST is supported."}`.

### 2. Point the page at the Worker

In `index.html`, find the SETTINGS block near the top of the script and set `WORKER_URL` to your Worker's full URL:

```js
const WORKER_URL = "https://epinetlens.<your-subdomain>.workers.dev";
```

### 3. Publish the page

Push `index.html` to a GitHub repository and turn on GitHub Pages for it (repository **Settings → Pages**). The app is then live at `https://<username>.github.io/<repo>/`.

### 4. Restrict the Worker to your site

Once you know the Pages address, edit this line in the Worker and redeploy:

```js
const ALLOWED_ORIGIN = "https://<username>.github.io";
```

Use the origin only: no repository path and no trailing slash.

## Configuration

All of these are constants at the top of `epinetlens-worker.js`. Redeploy the Worker after changing any of them.

| Setting | Default | What it controls |
| --- | --- | --- |
| `MODEL` | `claude-sonnet-5-5` | Which Claude model answers |
| `MAX_TOKENS` | `2000` | Maximum length of each reply |
| `MAX_MESSAGES` | `60` | Maximum number of turns accepted in one conversation |
| `MAX_TOTAL_CHARS` | `300000` | Maximum combined size of conversation and context |
| `ALLOWED_ORIGIN` | `*` | Which website may call the Worker |
| `WORKSHOP_CODE` | empty | Optional shared code (see below) |
| `SYSTEM_PROMPT` | EpinetLens prompt | The analyst's instructions and output format |

## Controlling access and cost

The Worker is a public web address, so anyone who finds it can use it at your expense. The safeguards, in rough order of strength:

- **Spend limit.** Set a monthly limit on the API key in the Anthropic Console. This is the only hard ceiling on cost.
- **Locked-down request.** The Worker builds the request itself, so callers cannot change the model, lift the length cap or replace the system prompt.
- **`ALLOWED_ORIGIN`.** Stops other websites from calling the Worker from a browser. It does not stop scripts.
- **`WORKSHOP_CODE`.** If set in the Worker, set the same value in `index.html`. It is visible in the page source, so it deters casual bots only.
- **Rate limiting.** Can be added in the Cloudflare dashboard if needed.

To shut off access for everyone at once, delete the `ANTHROPIC_API_KEY` secret or disable the Worker.

## Using the app

- **Start an analysis:** paste a text into the message box and press Enter (Shift+Enter for a new line).
- **Step through it:** EpinetLens presents its seven elements one at a time (Overview, Epinet Structure Narrative, Trust & Authentication Analysis, Strategic Moves, Robustness Assessment, Interpretive Consequences, Epinets-Lens Insight). Answer its question or type "continue".
- **Persistent Context:** background entered in the side panel and confirmed with **Set Context** is applied to every message that follows, until cleared.
- **Reset Conversation & Context:** clears both and starts over.

## Troubleshooting

| Message on the page | Likely cause |
| --- | --- |
| `Failed to fetch` | `WORKER_URL` is wrong, or `ALLOWED_ORIGIN` does not match the page's address |
| `Server is not configured with an API key.` | The `ANTHROPIC_API_KEY` secret is missing or misnamed |
| `Missing or incorrect workshop code.` | `WORKSHOP_CODE` differs between the Worker and `index.html` |
| An authentication or billing error from Anthropic | The key is invalid, revoked, or out of credit |
| A model-not-found error | `MODEL` is not a current model ID |
| `That reply hit the length limit.` | Type "continue", or raise `MAX_TOKENS` |

## Reference

Moldoveanu, M. C., & Baum, J. A. C. (2014). *Epinets: The Epistemic Structure and Dynamics of Social Networks*. Stanford University Press.
