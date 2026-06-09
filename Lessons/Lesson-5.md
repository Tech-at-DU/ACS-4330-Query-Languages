# ACS 4330 - Lab: Complete the Weather App

**Estimated time:** 2–3 hours

**Prerequisites:** Completed Lessons 3 and 4. You have a working Apollo Server and a React + Vite project.

<!-- > -->

## Goal

Finish the React + GraphQL Weather app before moving on to mutations in Lesson 6.

Use this lesson to close any gaps, fix bugs, and verify everything works end-to-end.

<!-- > -->

## Self-Assessment Checklist

Work through this list. Each item should be working before you move on.

<!-- > -->

### Server (from Lesson 3)

- [ ] Apollo Server starts with `npm start` at `http://localhost:4000`
- [ ] Apollo Sandbox loads at `http://localhost:4000`
- [ ] `getWeather(zip: 94122, units: imperial)` returns temperature and description
- [ ] `getWeather` accepts a `units` enum (`standard`, `metric`, `imperial`)
- [ ] Server returns `temperature`, `description`, `feels_like`, `temp_min`, `temp_max`, `pressure`, `humidity`
- [ ] Server returns `cod` and `message` when an invalid zip is provided
- [ ] `.env` file exists with `OPENWEATHERMAP_API_KEY` set
- [ ] `.gitignore` includes both `node_modules` and `.env`

<!-- > -->

### Client (from Lesson 4)

- [ ] Vite React app starts with `npm run dev` at `http://localhost:5173`
- [ ] `src/apolloClient.js` exists and exports `client` with `uri: 'http://localhost:4000/'`
- [ ] `src/main.jsx` wraps `<App>` in `<ApolloProvider>`
- [ ] Weather component has a form with a zip code input
- [ ] Submitting the form calls the GraphQL server and loads weather data
- [ ] Weather data displays after a successful query
- [ ] Invalid zip code shows an error message (not a crash)
- [ ] A dedicated component displays the weather data (receives data as props)

<!-- > -->

## Common Problems

These are the most common issues students hit when putting the server and client together.

<!-- > -->

### "Network Error" or "Failed to fetch"

**Cause:** Server isn't running, or wrong URI in `apolloClient.js`.

**Fix:**
1. Confirm your server is running: open `http://localhost:4000` in a browser
2. Check `uri` in `src/apolloClient.js` — must be `'http://localhost:4000/'` (with trailing slash, no `/graphql`)

<!-- > -->

### Apollo Sandbox works but React app gets an error

**Cause:** Apollo Server 4 standalone uses CORS-safe headers in the browser but sometimes needs explicit configuration.

**Fix:** If you see CORS errors in the browser console, update your server to use Apollo Server with Express middleware (covered in Lesson 12). For now, confirm the URI is correct first.

<!-- > -->

### Temperature comes back as a huge number (e.g. 295)

**Cause:** OpenWeatherMap defaults to Kelvin (`standard` units).

**Fix:** Add `units: imperial` or `units: metric` to your query, and default the resolver to `'imperial'` if no unit is passed.

<!-- > -->

### `weather.data.getWeather` is null or undefined

**Cause:** The zip code was invalid, so the API returned an error response.

**Fix:** Check `cod` and `message` in the response. Make sure those fields are in your GraphQL schema and your resolver returns them.

<!-- > -->

### State update after `client.query()` doesn't re-render

**Cause:** You're storing the raw Apollo response object (which contains `data`, `loading`, `errors`). The template may expect `weather.data.getWeather.temperature`.

**Fix:** Either store the full response and access `weather.data.getWeather.temperature`, or extract only what you need before calling `setWeather`.

<!-- > -->

## Challenges

Work through these if you haven't already. They're from Lesson 4 — complete as many as possible before Lesson 6.

<!-- > -->

Refer to [Lesson 4](Lesson-4.md) for details on each:

- **Challenge 5** — Handle data with conditional rendering
- **Challenge 6** — Make a component to display the weather
- **Challenge 7** — Handle errors (invalid zip)
- **Challenge 8** — Style your work
- **Challenge 9** — Use units (radio buttons for metric/imperial)
- **Challenge 10** — Get current location from the browser
- **Challenge 11** — Add weather-by-location to your GraphQL API
- **Challenge 12** — Fetch weather by geolocation from React

<!-- > -->

## Stretch: Enable HTTPS Locally

The browser's Geolocation API requires HTTPS. If you're working on Challenges 10–12, you'll need HTTPS running locally.

<!-- > -->

### Vite — Enable HTTPS

Install the Vite mkcert plugin:

```bash
npm install -D vite-plugin-mkcert
```

Update `vite.config.js`:

```js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import mkcert from 'vite-plugin-mkcert'

export default defineConfig({
  plugins: [react(), mkcert()],
  server: { https: true }
})
```

Your React app will now run at `https://localhost:5173`.

<!-- > -->

### Apollo Server — Enable HTTPS

Apollo Server 4 standalone doesn't expose an HTTPS option directly. For local HTTPS on the server, the simplest approach is to use a reverse proxy like [Caddy](https://caddyserver.com) or switch to the Express middleware integration (covered in Lesson 12).

For geolocation challenges, a common workaround: serve only the **client** over HTTPS (Vite above). The server can stay HTTP on localhost since the browser treats `localhost` as a secure context for most purposes in Chrome and Firefox.

<!-- > -->

## After This Lesson

Before moving to Lesson 6 (Mutations), your app should pass every item in the server and client checklists above.

If you're stuck on something, note the specific error message and bring it to the weekly check-in.
