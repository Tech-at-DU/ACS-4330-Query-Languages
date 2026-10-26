# ACS 4330 Advanced - Week 7: Deployment and Final Presentation

**Estimated time:** 3–4 hours to deploy + 1 hour to prepare your presentation

**Prerequisites:** Completed Weeks 1–6. Your final project works locally.

<!-- > -->

## Learning Outcomes

By the end of this lesson, you should be able to:

1. Prepare a GraphQL server and a React client for production
2. Deploy an Apollo Server to a Node hosting platform
3. Deploy a Vite + React client to a static hosting platform
4. Configure environment variables, CORS, and WebSocket URLs for production
5. Debug a deployed app using platform logs and the browser's developer tools

<!-- > -->

## Review

From memory:

- What's in your server's `.env` file? Why isn't it committed?
- What URL does your client use to reach the server? Where is it set?
- Why does the WebSocket link use `ws://` instead of `http://`?

<!-- > -->

## The Big Picture

You'll deploy three things:

```
Browser
  │
  ├── https://your-app.vercel.app        ← React client (static files)
  │
  └── https://your-api.onrender.com/graphql
        ├── HTTP  (queries, mutations)   ← Apollo Server
        └── WSS   (subscriptions)
              │
              └── MongoDB Atlas          ← already in the cloud
```

Your database is already hosted on Atlas. The server and client still run only on your laptop.

This lesson uses **Render** for the server and **Vercel** for the client. **Railway** (server) and **Netlify** (client) work almost the same way. All four have free tiers.

<!-- > -->

# Part 1 — Prepare for Production

Do these steps locally, test them, and push before you deploy anything.

<!-- > -->

## Server Checklist

**1. A `start` script.** Hosts run `npm start`. In `server/package.json`:

```json
"scripts": {
  "start": "node server.js",
  "dev": "node --watch server.js"
}
```

**2. A Node version.** Tell the host which Node to use:

```json
"engines": {
  "node": ">=20.19"
}
```

**3. Read `PORT` from the environment.** You did this in Week 5:

```js
const PORT = process.env.PORT || 4000
```

The host assigns the port. If you hard-code `4000`, the deploy fails or hangs.

**4. No secrets in code.** Search your code for your Mongo password and JWT secret. They should only appear in `.env`, which is gitignored.

<!-- > -->

## Lock Down CORS

`cors()` with no options allows **any** website to call your API. In production, allow only your client:

```js
app.use(
  '/graphql',
  cors({ origin: process.env.CLIENT_ORIGIN?.split(',') ?? true }),
  express.json(),
  expressMiddleware(server, { /* ... */ })
)
```

- Locally, `CLIENT_ORIGIN` isn't set, so `origin: true` allows everything
- In production, set `CLIENT_ORIGIN=https://your-app.vercel.app`. Separate multiple origins with commas
- No trailing slash: `https://your-app.vercel.app`, not `https://your-app.vercel.app/`

<!-- > -->

## Client Checklist

**Move the server URLs into environment variables.** Vite exposes variables that start with `VITE_` as `import.meta.env.VITE_...`.

`client/.env.development`:

```
VITE_GRAPHQL_HTTP=http://localhost:4000/graphql
VITE_GRAPHQL_WS=ws://localhost:4000/graphql
```

`src/apolloClient.js`:

```js
const httpLink = new HttpLink({ uri: import.meta.env.VITE_GRAPHQL_HTTP })

const wsLink = new GraphQLWsLink(
  createClient({
    url: import.meta.env.VITE_GRAPHQL_WS,
    connectionParams: () => ({ authorization: authHeader() }),
  })
)
```

Run `npm run dev` and confirm everything still works.

<!-- > -->

> **`VITE_` variables are public.** They're compiled into the JavaScript that ships to every browser. Server URLs are fine there. **API keys and secrets are not.**

> **`VITE_` variables are baked in at build time.** If you change one in your hosting dashboard, you have to **redeploy** the client.

<!-- > -->

**Build locally first:**

```bash
npm run build
npm run preview
```

If `npm run build` fails on your laptop, it will fail on the host too. Fix it here, where the errors are easier to read.

<!-- > -->

## Atlas Network Access

Atlas only accepts connections from IP addresses on its allow list. Hosting platforms don't give you a fixed IP address on their free tiers, so:

1. Atlas → **Network Access** → **Add IP Address**
2. Choose **Allow Access from Anywhere** (`0.0.0.0/0`)

Your database is still protected by its username and password. Use a long, random password for the database user.

<!-- > -->

# Part 2 — Deploy the Server (Render)

<!-- > -->

## Create the Web Service

1. Push your code to GitHub
2. Sign in to [render.com](https://render.com) with GitHub
3. **New** → **Web Service** → select your repo
4. Configure:

| Setting | Value |
|---|---|
| Root Directory | `final-project/server` (wherever your server's `package.json` is) |
| Runtime | Node |
| Build Command | `npm install` |
| Start Command | `npm start` |
| Instance Type | Free |

<!-- > -->

## Environment Variables

In the service's **Environment** tab, add:

| Key | Value |
|---|---|
| `MONGO_URI` | Your Atlas connection string |
| `JWT_SECRET` | A **new** long random string (not the one in your local `.env`) |
| `NODE_ENV` | `production` |

Don't add `CLIENT_ORIGIN` yet. You'll add it after deploying the client. (Don't add it with an empty value: an empty origin blocks every browser.)

Click **Deploy**. Watch the **Logs** tab until you see `Connected to MongoDB` and `Server ready`.

<!-- > -->

## Test the Server

Your server URL looks like `https://your-api.onrender.com`. Apollo Server is at `/graphql`.

With `NODE_ENV=production`, Apollo Server **turns off introspection**, so you can't explore the schema in Sandbox. That's on purpose: it stops strangers from mapping your whole API. Test with `curl` instead:

```bash
curl -X POST https://your-api.onrender.com/graphql \
  -H "Content-Type: application/json" \
  -d '{"query":"{ __typename }"}'
```

You should get `{"data":{"__typename":"Query"}}`.

> **Free tier sleep:** Render's free services go to sleep after about 15 minutes with no traffic. The first request after that takes 30–60 seconds while the server wakes up. Open your server URL a minute before your demo.

<!-- > -->

# Part 3 — Deploy the Client (Vercel)

<!-- > -->

## Create the Project

1. Sign in to [vercel.com](https://vercel.com) with GitHub
2. **Add New** → **Project** → import your repo
3. Configure:

| Setting | Value |
|---|---|
| Root Directory | `final-project/client` |
| Framework Preset | Vite |
| Build Command | `npm run build` (default) |
| Output Directory | `dist` (default) |

<!-- > -->

## Environment Variables

Add:

| Key | Value |
|---|---|
| `VITE_GRAPHQL_HTTP` | `https://your-api.onrender.com/graphql` |
| `VITE_GRAPHQL_WS` | `wss://your-api.onrender.com/graphql` |

Note **`wss://`**, the secure version of `ws://`. A page served over `https://` isn't allowed to open an insecure `ws://` connection.

Click **Deploy**.

<!-- > -->

## Connect the Two

1. Copy your Vercel URL (e.g. `https://your-app.vercel.app`)
2. In Render, set `CLIENT_ORIGIN` to that URL. Render redeploys automatically
3. Open your Vercel URL

<!-- > -->

# Part 4 — Smoke Test

<!-- > -->

## Test Everything in Production

Open the deployed app with DevTools open. Go through this list:

- [ ] The page loads with no console errors
- [ ] Lists load data from the production database
- [ ] Register a **new** account (production has its own users)
- [ ] Log in, refresh the page, and confirm you're still logged in
- [ ] Every create/update/delete updates the UI without a refresh
- [ ] Protected actions fail when logged out
- [ ] Pagination works
- [ ] Subscriptions: open two windows and confirm live updates (if you built subscriptions)
- [ ] Log out and confirm private data disappears

<!-- > -->

## When It Doesn't Work

| Symptom | Likely cause | Where to look |
|---|---|---|
| Server deploy fails | Missing `start` script, wrong root directory, build error | Render **Logs** |
| `MongoServerSelectionError` | Atlas Network Access doesn't allow `0.0.0.0/0`, or a typo in `MONGO_URI` | Render **Logs** |
| Browser: "blocked by CORS policy" | `CLIENT_ORIGIN` doesn't exactly match the client URL | Browser console; Render env vars |
| Client calls `localhost:4000` | `VITE_` variables not set, or set after the build | Vercel env vars, then **Redeploy** |
| WebSocket fails to connect | Using `ws://` instead of `wss://`, or the wrong path | Browser **Network → WS** |
| Everything is slow the first time | Free tier was asleep | Wait 60 seconds and retry |
| Works locally, `undefined` in production | Variable missing in the hosting dashboard | Compare `.env` with dashboard keys |

**Always read the server logs first.** Most deploy problems show up there as a clear error.

<!-- > -->

# Part 5 — Final Presentation

<!-- > -->

## Presentation (10 minutes)

Present at your final check-in. No slides; use your deployed app and your code.

1. **Demo (3 min)** — Use the **deployed** app. Register, log in, and use the main features
2. **Schema (2 min)** — Walk through your types and one relationship. Explain one embed-vs-reference decision
3. **One feature deep-dive (3 min)** — Pick one: your auth rules, your pagination, your subscription, or your DataLoader. Show the code and explain why it's built that way
4. **Reflection (2 min)** — What was hardest? What would you change with another week?

<!-- > -->

## Final Submission

Before your final check-in:

1. Push all code to `final-project/` in your course repo
2. Update `final-project/README.md` with:
   - Live client URL and server URL
   - A test account (email + password) your instructor can use
   - Which requirements and stretch goals you completed
   - How to run it locally (env variables needed, with **example** values only)
3. Paste your repo link and live URL in the course tracker

See the [final project spec](../Assignments/FinalProjectSpec.md) for the full requirements and rubric.

<!-- > -->

## After This Lesson

- Server and client deployed and smoke-tested
- `final-project/README.md` complete
- Links pasted in the course tracker
- Presentation given at your final check-in

<!-- > -->

### Evaluate Your Work

| | Does not meet expectations | Meets expectations | Exceeds expectations |
|:---:|:---:|:---:|:---:|
| Server deploy | Not deployed or crashes | Deployed; connects to Atlas; secrets in env vars | CORS limited to the client origin; separate production JWT secret |
| Client deploy | Not deployed or calls localhost | Deployed; talks to the production server | Server URLs come from `VITE_` env vars; subscriptions work over `wss://` |
| Smoke test | Major features broken in production | All required features work in production | Full checklist passes, including logout and protected actions |
| Documentation | No README | README has URLs and a test account | README explains local setup and lists completed features |

<!-- > -->

## Resources

- https://render.com/docs/deploy-node-express-app
- https://render.com/docs/environment-variables
- https://vercel.com/docs/frameworks/vite
- https://vite.dev/guide/env-and-mode
- https://www.mongodb.com/docs/atlas/security/ip-access-list/
- https://www.apollographql.com/docs/apollo-server/security/cors
- https://docs.railway.com and https://docs.netlify.com — if you use these instead
