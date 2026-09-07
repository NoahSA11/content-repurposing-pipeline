# Content Repurposing Pipeline

Paste in one long-form piece of writing, get it repurposed into a LinkedIn post,
an X thread, and an email blurb — generated in parallel by an n8n workflow
calling OpenRouter's free-tier LLM API.

**Live demo:** https://content-repurposing-pipeline-noah-adler-s-projects.vercel.app

> This demo runs against a self-hosted n8n instance exposed through a
> Cloudflare Tunnel. It's only reachable while that instance is running — if
> the live demo doesn't respond, [insert demo GIF/recording here] shows it
> working end-to-end.

## How it works

```
index.html  --POST-->  n8n Webhook
                          |
             +------------+------------+
             |            |            |
        OR - LinkedIn  OR - X Thread  OR - Email     (parallel OpenRouter calls)
             |            |            |
             +------------+------------+
                          |
                        Merge (join, by position)
                          |
                     Shape Output (Set node)
                          |
                   Respond to Webhook
                          |
                    { linkedin, x_thread, email }
```

- **Frontend** (`index.html`): single static page, no build step. Textarea in,
  three styled output cards out, with copy-to-clipboard buttons.
- **Backend** (`n8n-workflow/content-repurposing-openrouter.json`): an n8n
  workflow — a Webhook trigger fans out to three parallel HTTP Request nodes,
  each hitting OpenRouter's OpenAI-compatible chat-completions API
  (`minimax/minimax-m3:free`) with a distinct system prompt per platform. A
  Merge node joins the three branches once they've all finished, then a Set
  node reads each branch's result directly off its source node (avoids
  relying on the Merge node's own output shape) and shapes the final JSON
  response.

## Running your own copy

1. Import `n8n-workflow/content-repurposing-openrouter.json` into your own
   n8n instance.
2. Get a free API key at [openrouter.ai/keys](https://openrouter.ai/keys) (no
   card required for `:free`-suffix models) and paste it into the
   `Authorization` header of each of the three HTTP Request nodes.
3. Activate the workflow, then expose it publicly (e.g. `cloudflared tunnel
   --url http://localhost:5678`) or host n8n somewhere public.
4. Set `WEBHOOK_URL` at the top of the `<script>` block in `index.html` to
   your workflow's production webhook URL.
5. Deploy `index.html` anywhere that serves static files (Vercel, Netlify,
   GitHub Pages, etc.) — no build step required.

## Stack

n8n · OpenRouter (`minimax/minimax-m3:free`) · vanilla HTML/CSS/JS · Vercel
