# Putting n8n on Your Own Domain — A Complete Beginner's Guide

This guide takes you from **zero** to **your n8n running live at your own web
address** (like `n8n.yourdomain.com`), accessible from any device, anywhere.

**No prior experience needed.** Every step is explained in plain English.

---

## What you're about to build (in plain English)

Right now your n8n runs only on your own computer at `http://localhost:5678`.
"localhost" means *this machine only* — nobody else can reach it.

A **Cloudflare Tunnel** creates a secure, private pipe from Cloudflare's network
straight into your computer. When someone visits `n8n.yourdomain.com`, Cloudflare
catches that request and pushes it down the tunnel to the n8n running on your
machine. The visitor sees your n8n; they never see your real IP address or get
into anything else on your computer.

**Why this is great for beginners:**

- ✅ **Free** (Cloudflare's tunnel is free; you only pay for the domain name).
- ✅ **No server to rent** — your own computer is the server.
- ✅ **No router setup** — you don't open ports or expose your home network.
- ✅ **Automatic HTTPS** — the little padlock 🔒 is handled for you.

**What you'll need:**

| Thing | Cost | Notes |
| ----- | ---- | ----- |
| A domain name | ~$5–15 / year | e.g. `yourdomain.com` |
| A Cloudflare account | Free | |
| This project running locally | Free | See the main [README](README.md) first |

⏱️ **Time:** about 30–45 minutes, plus some waiting for the domain to activate.

> ⚠️ **Before you start:** make sure your local n8n already works by following the
> main [README](README.md). You should be able to open `http://localhost:5678`
> and log in. This guide only adds the "put it online" part on top.

---

## Step 1 — Get a domain name

A domain is your web address, like `yourdomain.com`. You rent it yearly from a
"registrar".

**The easiest path is to buy it directly from Cloudflare** (then Steps 2–3 are
basically already done for you):

1. Create a free account at **https://dash.cloudflare.com/sign-up**.
2. In the dashboard, go to **Domain Registration → Register Domains**.
3. Search for a name you like, pick one, and pay. Cloudflare sells domains at
   cost (no markup).

> **Already own a domain elsewhere** (GoDaddy, Namecheap, Google Domains, etc.)?
> No problem — keep it where it is and follow Step 3 to connect it to Cloudflare.

**Don't have a domain and want the simplest possible experience?** Buy it from
Cloudflare. It removes an entire step.

---

## Step 2 — Create your free Cloudflare account

If you bought your domain from Cloudflare, you already have this — skip ahead.

Otherwise:

1. Go to **https://dash.cloudflare.com/sign-up**.
2. Sign up with your email and a strong password.
3. Verify your email when Cloudflare sends you a confirmation link.

That's it. The account is free forever.

---

## Step 3 — Connect your domain to Cloudflare

**Skip this step if you bought your domain from Cloudflare** — it's already
connected.

If your domain is registered somewhere else, you need to tell the internet to
route it through Cloudflare. This is a one-time setup.

1. In the Cloudflare dashboard, click **Add a site** (or **+ Add → Existing
   domain**).
2. Type your domain (e.g. `yourdomain.com`) and click **Continue**.
3. Choose the **Free** plan.
4. Cloudflare scans your existing settings and shows you **two nameservers**,
   something like:

   ```
   dana.ns.cloudflare.com
   rob.ns.cloudflare.com
   ```

5. Now go to **the website where you bought the domain** (GoDaddy, Namecheap,
   etc.), find the **Nameservers / DNS** settings for your domain, and **replace**
   the existing nameservers with the two Cloudflare gave you.

   > This is the "change of address" step — it tells the world "Cloudflare is now
   > in charge of this domain."

6. Back in Cloudflare, click **Done, check nameservers**.

⏳ **Now wait.** Cloudflare needs the change to take effect. This can take from a
few minutes up to a few hours (occasionally up to 24h). Cloudflare emails you when
your domain is **Active**. Don't continue until it says Active.

---

## Step 4 — Open the Zero Trust dashboard

The tunnel feature lives in a section of Cloudflare called **Zero Trust** (it's
free for personal use).

1. Go to **https://one.dash.cloudflare.com/**
   *(or, in the main dashboard, click **Zero Trust** in the left sidebar).*
2. The very first time, it may ask you to pick a **team name** (any name) and
   choose the **Free** plan. If it asks for a payment card to verify, that's
   normal for the free plan — you won't be charged.

---

## Step 5 — Create the tunnel

1. In the Zero Trust dashboard, open **Networks → Tunnels** in the left sidebar.
2. Click **Create a tunnel**.
3. Choose **Cloudflared** as the connector type, then **Next**.
4. Give your tunnel a name — for example `n8n-home` — and click **Save tunnel**.

You'll land on an "Install and run a connector" page. **You can ignore the install
commands it shows** — this project already runs `cloudflared` for you inside
Docker. The only thing you need from this page is the **token**.

### Get your token

On that install page, Cloudflare shows a command that contains a very long string.
It looks like:

```
cloudflared service install eyJ...A-VERY-LONG-STRING-UNIQUE-TO-YOU...In0
```

The long part after `install` (starting with `eyJ...`) **is your token**. Copy
**only that long string**. Keep it secret — anyone with it can run your tunnel.

> 🔒 You'll paste this into your `.env` file in Step 7. It never goes into the
> public code — `.env` is ignored by git on purpose.

Click **Next** to continue to the routing step.

---

## Step 6 — Point your subdomain at n8n

Now you tell the tunnel *which web address* should open *which local service*.

On the **Public Hostnames** step (or later via the tunnel's **Public Hostname**
tab), fill in:

| Field | What to enter | Example |
| ----- | ------------- | ------- |
| **Subdomain** | the part before your domain | `n8n` |
| **Domain** | pick your domain from the dropdown | `yourdomain.com` |
| **Path** | leave blank | |
| **Type** (Service) | choose **HTTP** | `HTTP` |
| **URL** (Service) | type exactly this | `localhost:5678` |

So the result is: **`n8n.yourdomain.com`** → **`HTTP`** → **`localhost:5678`**.

Click **Save**.

> **Why `localhost:5678`?** Inside this project, the `cloudflared` container is set
> up to share the network with n8n, so to it, n8n really is at `localhost:5678`.
> You don't need to change this — it's already wired up in `docker-compose.yml`.

---

## Step 7 — Put your settings into the project

On your computer, open the `.env` file in your project folder (the same folder as
`docker-compose.yml`). If you don't have a `.env` yet, create one by copying the
example:

```bash
cp .env.example .env
```

Add or update these lines (replace `yourdomain.com` and the token with your own):

```env
# Your public web address
N8N_HOST=n8n.yourdomain.com
N8N_PROTOCOL=https
N8N_WEBHOOK_URL=https://n8n.yourdomain.com/

# Turn the tunnel on automatically
COMPOSE_PROFILES=tunnel

# Paste the long token from Step 5 here
CLOUDFLARE_TUNNEL_TOKEN=eyJ...PASTE-YOUR-OWN-LONG-TOKEN-HERE...In0
```

Save the file.

> 💡 `COMPOSE_PROFILES=tunnel` is the on/off switch. With it, `docker compose up`
> automatically starts the tunnel. Remove it (or comment it out with a `#`) to go
> back to local-only.

---

## Step 8 — Start it up

In your terminal, from the project folder:

```bash
docker compose up -d
```

Docker starts three things now: your **database**, **n8n**, and the **tunnel**.

Check that the tunnel connected:

```bash
docker compose logs cloudflared | grep -i "registered tunnel connection"
```

If you see one or more lines saying **"Registered tunnel connection"**, you're
connected to Cloudflare. 🎉

---

## Step 9 — Visit your site

Open your browser and go to:

```
https://n8n.yourdomain.com
```

You should see your n8n, with a secure 🔒 padlock. **You're live on the internet.**

> First visit not loading? Give it 1–2 minutes — DNS for the new subdomain can take
> a moment the first time. Then refresh. See Troubleshooting below if it persists.

---

## Step 10 (Important) — Lock it down

Your n8n is now reachable by **anyone on the internet**, so the login page is
public. Do these before relying on it:

### A. Use strong passwords

In your `.env`, make sure these are NOT `admin` / `change_me` / `root`:

```env
POSTGRES_PASSWORD=use_a_long_random_password
N8N_BASIC_AUTH_PASSWORD=use_a_different_long_password
```

After changing them, run `docker compose up -d` again.

> Note: changing `POSTGRES_PASSWORD` only takes effect on a brand-new database.
> If you already have data, change it inside the database instead, or start fresh
> with `docker compose down -v` (this erases data).

### B. Add Cloudflare Access (highly recommended)

This puts a login wall **in front** of n8n, so only YOU can even see it. Only
people you allow get a one-time PIN by email; everyone else is blocked before they
ever reach n8n.

1. In **Zero Trust → Access → Applications**, click **Add an application**.
2. Choose **Self-hosted**.
3. **Application name:** anything (e.g. `n8n`).
4. **Application domain:** `n8n` + `yourdomain.com`.
5. Click **Next** to add a policy:
   - **Policy name:** `Only me`
   - **Action:** Allow
   - **Include → Emails →** your email address.
6. Save. Now visiting `n8n.yourdomain.com` first asks for your email, sends a PIN,
   and only then shows n8n.

This is the single biggest security upgrade for a self-hosted tool on the open
internet. Strongly recommended.

---

## Turning the tunnel off

To take n8n back offline (local-only) at any time:

1. In your `.env`, comment out the switch:
   ```env
   #COMPOSE_PROFILES=tunnel
   ```
2. Apply it:
   ```bash
   docker compose up -d
   ```

n8n keeps running locally at `http://localhost:5678`; it's just no longer on the
internet.

---

## Troubleshooting

**"This site can't be reached" / the page never loads**
- Make sure the containers are running: `docker compose ps` (you should see
  `n8n`, `postgres`, and `cloudflared`).
- Check the tunnel connected: `docker compose logs cloudflared | tail -20`.
  Look for "Registered tunnel connection". If you see token errors, re-check the
  token in `.env`.
- Confirm the public hostname in Cloudflare points to **`HTTP` → `localhost:5678`**.

**Error 1033 / "Argo Tunnel error" / "Bad Gateway 502"**
- The tunnel is up but can't reach n8n. Make sure n8n is running
  (`docker compose ps`) and the service URL in Cloudflare is exactly
  `localhost:5678` with type **HTTP** (not HTTPS).

**Domain still shows "Pending" in Cloudflare**
- The nameserver change (Step 3) hasn't finished. Wait longer, and double-check you
  replaced the nameservers correctly at your registrar.

**Can log in locally but not on the domain (or vice versa)**
- This project sets `N8N_SECURE_COOKIE=false` so both work. If you changed that,
  set it back in `docker-compose.yml` and run `docker compose up -d`.

**I changed `.env` but nothing changed**
- Run `docker compose up -d` (not `start`). Compose only re-reads `.env` when it
  recreates the containers.

---

## How it fits together (the big picture)

```
   Your browser                 Cloudflare's network            Your computer
   ────────────                 ────────────────────            ─────────────
  n8n.yourdomain.com  ──HTTPS──▶   (catches request)  ──tunnel──▶  cloudflared
                                                                       │
                                                                       ▼
                                                                  n8n :5678
                                                                       │
                                                                       ▼
                                                                 postgres (data)
```

- **Cloudflare** handles the public address and the HTTPS padlock.
- **cloudflared** (a container in this project) holds the secure tunnel open.
- **n8n** + **postgres** are your actual app and its data, never directly exposed.

That's the whole pattern. Anyone who follows these steps with their own domain and
token gets the exact same setup running on their own machine.
