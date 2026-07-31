# The Chat App Master Roadmap
### Vanilla JS → Supabase → React, with Auth right after core chat

**Tech stack:** HTML/CSS → Vanilla JS → Supabase (DB, Realtime, Auth, Storage) → React (Vite) → Vercel deployment

**Rule for every stage:** No copy-paste implementations. Every stage gives you a Goal, a numbered task list, the exact API surface you'll need, a search query if you're stuck, and a verification step to prove it actually works before moving on. Code snippets only appear where an API's syntax is genuinely new and non-obvious (e.g. Supabase's realtime subscribe signature) — the logic is always yours to write.

**Order of versions:**
1. Version 1 — HTML & CSS UI Polish
2. Version 2a — Vanilla JS + Supabase: Core Realtime Chat
3. Version 2b — Authentication (Supabase Auth)
4. Version 3 — The React Refactor
5. Version 4 — Rich Messaging Features
6. Version 5 — Security, Polish & Deployment

---

# VERSION 1 — HTML & CSS UI Polish

**Overall goal:** Master layout structure, responsive design, and CSS variables using your existing `index.html` / `styles.css`. No JavaScript yet.
**Estimated time:** ~1 week, 10 hours.

## Stage 1.1 — Map the Existing Structure
**Goal:** Know exactly what every part of your current markup does before you touch anything.

**Tasks:**
1. Open `index.html`. Above every top-level element (sidebar, header, message list, input bar, individual message bubble), write a one-line HTML comment describing its job.
2. Open DevTools → Elements. Click each major section, confirm the box model (margin/border/padding) in the Computed panel matches what you expect.
3. Open DevTools → Coverage tab (Cmd/Ctrl+Shift+P → "Show Coverage"), reload the page, and identify CSS rules that are never applied. Delete dead CSS.
4. List every class name currently in `styles.css` in a scratch file, grouped by which section of the UI they style. This becomes your naming reference so you don't create duplicate/conflicting classes later.

**API surface:** None — pure inspection stage.

**Verification:** Without opening the file, you can describe the DOM tree from `<body>` down to a single chat bubble, and you can name which CSS class controls which visual property for at least 10 elements.

---

## Stage 1.2 — Mobile Responsiveness
**Goal:** Sidebar collapses into a slide-in overlay below 768px width, toggled by a hamburger button.

**Tasks:**
1. Add a `@media (max-width: 768px)` block in `styles.css`.
2. Inside it, change the sidebar's positioning: `position: fixed`, full height, `transform: translateX(-100%)`, and add `transition: transform 0.3s ease` on the base (non-media-query) sidebar rule so the transition applies both directions.
3. Add a hamburger `<button>` element in your header markup. Give it `display: none` by default, `display: block` inside the same media query.
4. Create a `.sidebar-open` class that sets `transform: translateX(0)` — this will be toggled by JS in Version 2, but for now you're just defining the CSS contract.
5. Add a semi-transparent `.sidebar-backdrop` div (initially `display: none`, later toggled) so tapping outside the sidebar will eventually close it.

**API surface:** None yet — CSS only.

**Search query if stuck:** `"CSS slide-in sidebar overlay mobile menu transform translateX transition"`

**Verification:** Manually add the class `sidebar-open` to the sidebar element in DevTools. It should slide in smoothly from off-screen with no layout jump, no horizontal scrollbar appearing, and no content behind it shifting.

---

## Stage 1.3 — Dark Theme via CSS Custom Properties
**Goal:** A toggleable dark/light theme controlled entirely by CSS variables, no JS logic beyond a class swap.

**Tasks:**
1. In `:root`, define variables for every color currently hardcoded in your stylesheet: `--bg-primary`, `--bg-secondary`, `--text-primary`, `--text-secondary`, `--accent`, `--border-color`, `--bubble-sent-bg`, `--bubble-received-bg`, etc.
2. Replace every hardcoded hex/rgb value in the rest of the stylesheet with `var(--variable-name)`.
3. Create a `.dark-theme` class (intended to sit on `<body>` or `<html>`) that redefines the same variable names with dark-mode values.
4. Grep your CSS file for any remaining raw hex codes (`#` followed by 3 or 6 hex digits) to catch stragglers you missed.
5. Pick an accent color for dark mode that maintains at least 4.5:1 contrast against your dark background — use a contrast checker if unsure.

**API surface:** None — CSS only.

**Verification:** Add `.dark-theme` to `<body>` manually in DevTools. Every visible surface — background, text, borders, bubbles — should re-theme correctly with zero elements left in the wrong color scheme.

---

## Stage 1.4 — Component-Level Polish
**Goal:** Chat bubbles, timestamps, and the input bar look intentional, not default-browser.

**Tasks:**
1. Style sent vs. received bubbles distinctly: different background color, alignment (`margin-left: auto` vs `margin-right: auto` or flexbox `align-self`), and border-radius (asymmetric radius on the "tail" corner is a common pattern — try it).
2. Add a timestamp element under/beside each bubble, styled small and low-contrast (`font-size: 0.75rem`, muted color variable).
3. Style the input bar: rounded input field, a distinct send button, proper vertical alignment using flexbox (`align-items: center`).
4. Add a hover/focus state to the send button (`transform: scale(1.05)` or a background color shift) so it doesn't feel static.
5. Check text wrapping on long messages — `word-wrap: break-word` or `overflow-wrap: anywhere` so a long unbroken string (like a URL) doesn't blow out the bubble width. Set a `max-width` on bubbles (commonly 70-75% of container width).

**API surface:** None — CSS only.

**Verification:** Paste a very long single-word string (40+ characters, no spaces) into a mock bubble in your HTML. It should wrap inside the bubble, not overflow the container or the screen.

---

**Version 1 exit checklist:**
- [ ] Sidebar CSS contract for mobile is defined and manually toggleable
- [ ] Dark theme fully covers every visible color
- [ ] Bubbles, timestamps, and input bar look distinct and polished
- [ ] No hardcoded colors remain outside `:root`

---

# 🚨 COURSE TRIGGER: Scrimba Vanilla JavaScript Course
Focus specifically on: variables & scope, DOM selection (`querySelector`/`querySelectorAll`), event listeners (`click`, `keydown`, `submit`), array methods (`map`, `filter`, `forEach`), template literals, and `async`/`await` with `fetch`. You do not need to finish the entire course before starting Version 2a — once you're comfortable with DOM manipulation and event listeners, move on and learn `async/await` in parallel with Stage 2a.3.

---

# VERSION 2a — Vanilla JS + Supabase: Core Realtime Chat

**Overall goal:** Turn the static UI into a working chat where messages persist and multiple browser tabs/devices see new messages instantly, without auth yet (a plain text `user_name` field stands in for real identity for now).
**Estimated time:** ~1.5–2 weeks, 15–20 hours.

## Stage 2a.1 — Local Message Rendering (No Backend Yet)
**Goal:** Sending a message in the UI creates a new bubble on screen, purely client-side, before any database is involved.

**Tasks:**
1. Add a `click` event listener to the send button and a `keydown` listener on the textarea that checks for `Enter` (without `Shift`) to also trigger send.
2. On send: read the textarea's value, trim whitespace, and bail out early (do nothing) if the trimmed value is empty.
3. Write a function `createMessageBubble(text, isSent)` that builds the DOM structure for one bubble (using `document.createElement` or template literals + `innerHTML`) and appends it to `.messages-container`.
4. After appending, clear the textarea and set `messagesContainer.scrollTop = messagesContainer.scrollHeight` so the view jumps to the newest message.
5. Wire up the hamburger button from Stage 1.2: toggle the `.sidebar-open` class and the backdrop's visibility on click; clicking the backdrop should close the sidebar.

**API surface:**
- `element.addEventListener('click', handler)`
- `element.addEventListener('keydown', handler)` — check `event.key === 'Enter' && !event.shiftKey`
- `document.createElement`, `element.appendChild`, or `element.insertAdjacentHTML`
- `element.scrollTop`, `element.scrollHeight`

**Verification:** Typing a message and pressing Enter (or clicking Send) produces a new bubble at the bottom of the message list, the textarea clears, and the view auto-scrolls. Refreshing the page loses everything — that's expected, you haven't added persistence yet.

---

## Stage 2a.2 — Supabase Project & Table Setup
**Goal:** A live Supabase project with a `messages` table ready to receive data.

**Tasks:**
1. Create a free account at supabase.com and start a new project. Note your **Project URL** and **anon public API key** from Settings → API — you'll need both, but never commit them to a public repo without understanding RLS implications (covered in Version 5).
2. In the Table Editor, create a `messages` table with columns: `id` (int8, primary key, auto-increment), `user_name` (text), `text` (text), `created_at` (timestamptz, default `now()`).
3. Temporarily disable Row Level Security on this table (you'll re-enable and configure it properly in Version 5) so you can read/write freely while learning the basics — but be aware this means the table is world-writable for now; don't leave it this way once anyone else can find the URL.
4. Manually insert 2-3 test rows via the Table Editor UI to confirm the schema works before touching code.

**API surface:** Supabase dashboard only — no code yet.

**Verification:** You can see your manually-inserted test rows in the Table Editor, and you have your Project URL + anon key saved somewhere safe.

---

## Stage 2a.3 — Connect Supabase to Vanilla JS
**Goal:** Your page can read from and write to the `messages` table using the Supabase JS client loaded via CDN.

**Tasks:**
1. Add the Supabase JS CDN script tag to `index.html`, before your own script file.
2. Initialize the client using your Project URL and anon key. This is the one place a tiny snippet is worth showing since the initialization syntax is not guessable:

```javascript
const { createClient } = supabase;
const client = createClient('YOUR_PROJECT_URL', 'YOUR_ANON_KEY');
```

3. Write a function `sendMessageToDb(userName, text)` that calls `client.from('messages').insert({ user_name: userName, text })` and logs any error returned.
4. Write a function `fetchMessageHistory()` that calls `client.from('messages').select('*').order('created_at', { ascending: true })` and returns the array of rows.
5. On page load, call `fetchMessageHistory()` and render each row through your existing `createMessageBubble` function from Stage 2a.1.
6. Wire your Stage 2a.1 send handler to also call `sendMessageToDb` — decide whether you render the bubble immediately (optimistic UI) or wait for the DB write to confirm before rendering (you'll revisit this tradeoff in Stage 2a.4).

**API surface:**
- `client.from(table).select(columns)`
- `client.from(table).insert(rowObject)`
- `.order(column, { ascending: bool })`
- Both return a promise resolving to `{ data, error }` — always check `error` first

**Search query if stuck:** `"supabase-js insert select example vanilla javascript CDN"`

**Verification:** Refreshing the page shows your previously-sent messages loaded from the database, in the correct chronological order. Sending a new message and then manually refreshing shows it persisted too.

---

## Stage 2a.4 — Realtime Subscriptions
**Goal:** A message sent from one browser tab/device appears in another open tab within roughly a second, with no manual refresh.

**Tasks:**
1. Confirm Realtime is enabled for the `messages` table (Database → Replication in the Supabase dashboard — toggle the table on if it isn't already).
2. Set up a channel subscription that listens for `INSERT` events on the `messages` table. The subscribe syntax is the other spot worth a snippet since it's non-obvious:

```javascript
client
  .channel('messages-channel')
  .on('postgres_changes', { event: 'INSERT', schema: 'public', table: 'messages' },
    (payload) => { /* your handler: render payload.new as a bubble */ }
  )
  .subscribe();
```

3. In the handler, call your `createMessageBubble` function with the incoming row's data.
4. Now resolve the optimistic-UI question from Stage 2a.3: if you render locally on send AND the realtime event also fires for your own insert, you'll get duplicate bubbles. Pick one approach — either render only via the realtime subscription (simplest, slight delay) or render optimistically and skip rendering realtime events that match a message you just sent (track sent message IDs/timestamps to dedupe).
5. Test with two browser windows (or one normal + one incognito) side by side, sending from each.

**API surface:**
- `client.channel(name)`
- `.on('postgres_changes', filterObject, callback)`
- `.subscribe()`

**Search query if stuck:** `"supabase realtime postgres_changes INSERT event example"`

**Verification:** With two windows open side-by-side, a message sent in window A appears in window B within ~1 second with zero duplicate bubbles in either window.

---

## Stage 2a.5 — Loading, Error, and Empty States
**Goal:** The app doesn't look broken during the in-between moments — while messages are loading, if a send fails, or if there's no chat history yet.

**Tasks:**
1. Show a simple loading indicator (skeleton bubbles, a spinner, or a "Loading messages..." text) between page load and the resolution of `fetchMessageHistory()`.
2. If `fetchMessageHistory()` returns zero rows, show an empty state ("No messages yet — say hi!") instead of a blank container.
3. If `sendMessageToDb` returns an error, show a visible failure state (e.g. the bubble renders with a red "failed to send, tap to retry" indicator) rather than silently failing.
4. Disable the send button while a send is in-flight to prevent duplicate submissions from rapid double-clicks or double-Enter.

**API surface:** None new — this is UI/state handling around the functions you already wrote.

**Verification:** Throttle your network in DevTools (Network tab → Slow 3G) and reload — you should see the loading state, not a blank screen. Turn off WiFi and try sending — you should see a visible failure, not silence.

---

**Version 2a exit checklist:**
- [ ] Messages persist to Supabase and reload correctly on refresh
- [ ] Two open tabs see each other's messages in realtime with no duplicates
- [ ] Loading, empty, and error states are all handled visibly

---

# VERSION 2b — Authentication (Supabase Auth)

**Overall goal:** Replace the spoofable `user_name` text field with real accounts, so every message is tied to an actual authenticated user before you build anything else on top.
**Estimated time:** ~1 week, 8–10 hours.

## Stage 2b.1 — Supabase Auth Setup
**Goal:** Understand what Supabase Auth gives you and configure the project for email/password sign-in.

**Tasks:**
1. In the Supabase dashboard, go to Authentication → Providers and confirm Email is enabled (it is by default).
2. For development, turn off "Confirm email" under Authentication → Settings so you're not blocked by email verification while testing (re-enable, or configure a real email provider, before going live in Version 5).
3. Read through what a Supabase `session` object contains (user id, email, access token, expiry) — you don't need code yet, just understand the shape of the data you'll be working with.

**API surface:** Supabase dashboard only.

**Verification:** You can explain in one sentence what a "session" is versus a "user" in Supabase Auth terms.

---

## Stage 2b.2 — Sign Up and Log In Forms
**Goal:** A working sign-up form and login form, both calling Supabase Auth.

**Tasks:**
1. Build (or repurpose existing) HTML for a sign-up form (email, password, confirm password fields) and a login form (email, password), toggleable between the two.
2. Add client-side validation before calling the API: non-empty fields, password length minimum, passwords-match check on sign-up.
3. Wire the sign-up form to `client.auth.signUp({ email, password })` and the login form to `client.auth.signInWithPassword({ email, password })`.
4. Handle and display errors returned from each call (wrong password, email already registered, etc.) in a visible, non-console-only way.
5. On successful login, redirect/transition to the main chat view; on logout (add a logout button somewhere in the sidebar), call `client.auth.signOut()` and return to the login screen.

**API surface:**
- `client.auth.signUp({ email, password })`
- `client.auth.signInWithPassword({ email, password })`
- `client.auth.signOut()`

**Search query if stuck:** `"supabase auth signUp signInWithPassword vanilla js example"`

**Verification:** You can create a new account, get logged in automatically or via a subsequent login, log out, and log back in with the same credentials — all without touching the Supabase dashboard.

---

## Stage 2b.3 — Session Persistence & Route Guarding
**Goal:** Refreshing the page doesn't log you out, and the chat UI is unreachable without a valid session.

**Tasks:**
1. On page load, call `client.auth.getSession()` (or `getUser()`) before deciding what to render — show the login form if no session exists, show the chat UI if one does.
2. Subscribe to auth state changes with `client.auth.onAuthStateChange((event, session) => {...})` so login/logout happening in one tab is reflected without a manual reload (this mirrors what you built for messages in Stage 2a.4).
3. Since this is still a single-page vanilla JS app without a router, "guarding" means: keep both the auth UI and the chat UI in the DOM but toggle which is visible based on session state — don't let chat UI event listeners fire when there's no valid session.

**API surface:**
- `client.auth.getSession()`
- `client.auth.onAuthStateChange(callback)`

**Verification:** Log in, refresh the page — you're still logged in and see the chat, not the login form. Open DevTools, manually clear local storage, refresh — you're bounced back to the login form.

---

## Stage 2b.4 — Replace `user_name` With Real User Identity
**Goal:** Every message in the database is tied to the actual authenticated user, not a typed-in name.

**Tasks:**
1. Add a `user_id` column (uuid, references `auth.users`) to your `messages` table, replacing your reliance on the free-text `user_name` field for identity (you can keep a `display_name` derived from the user's profile if you want a friendly name shown in the UI — that's a Stage 4 concern, a raw email prefix is fine for now).
2. Update `sendMessageToDb` to pull the current user's id from the session (`client.auth.getUser()` or the session object you already have) and insert it as `user_id` instead of accepting a typed name.
3. Update your bubble rendering (`createMessageBubble`) to determine "sent vs received" styling by comparing the message's `user_id` to the current logged-in user's id, instead of a hardcoded assumption.
4. Backfill or discard your old test rows from Stage 2a.2 that don't have a valid `user_id` — they'll break your new rendering logic otherwise.

**API surface:** Same insert/select calls from Stage 2a.3, now including `user_id`.

**Verification:** Log in as two different accounts (two browser windows, two different emails). Messages sent from each account correctly show as "sent" in their own window and "received" in the other's — driven entirely by real user id, not a typed string.

---

**Version 2b exit checklist:**
- [ ] Sign up, log in, log out all work and show real errors on failure
- [ ] Session survives a page refresh; no session = no chat access
- [ ] Every message is tied to a real `user_id`, `user_name` text field is gone

---

# 🚨 COURSE TRIGGER: Scrimba React Basics Course
Focus on: JSX syntax, functional components, props, `useState`, `useEffect`, and conditional rendering. You'll lean heavily on `useEffect` for the realtime subscription pattern you already understand from Stage 2a.4.

---

# VERSION 3 — The React Refactor

**Overall goal:** Rebuild the same working app in React so the code is modular and scalable going forward. No new features here — the goal is identical behavior, better structure.
**Estimated time:** ~2 weeks, 20 hours.

## Stage 3.1 — Project Scaffolding
**Goal:** A running Vite + React project with Supabase reconnected.

**Tasks:**
1. Run `npm create vite@latest` and choose the React template.
2. Install the Supabase client via npm (`npm install @supabase/supabase-js`) instead of the CDN tag.
3. Create a `supabaseClient.js` file that exports a single initialized client instance — this becomes the one place your credentials live, imported everywhere else that needs it.
4. Move your Project URL and anon key into a `.env` file (Vite convention: `VITE_SUPABASE_URL`, `VITE_SUPABASE_ANON_KEY`), and add `.env` to `.gitignore` before your first commit in this new project.
5. Confirm the dev server runs and can successfully call `client.auth.getSession()` with no errors, proving the connection carried over correctly.

**API surface:**
- `import.meta.env.VITE_SUPABASE_URL` (Vite's env variable access pattern)

**Verification:** `npm run dev` starts cleanly, and a `console.log` of `getSession()` in your entry component returns without error (session will be null since you haven't rebuilt login yet — that's expected).

---

## Stage 3.2 — Component Architecture
**Goal:** Break the monolithic HTML/JS into components with clear, single responsibilities.

**Tasks:**
1. Sketch the component tree on paper first, before writing code: `App` → `AuthGate` → (`LoginForm` | `ChatLayout`), and `ChatLayout` → `Sidebar`, `ChatHeader`, `MessageList`, `MessageInput`.
2. Create each component as its own file returning static/placeholder JSX first — get the layout rendering correctly with fake data before wiring any real state or Supabase calls.
3. Decide, before writing state logic, which component owns which piece of data (e.g. `ChatLayout` likely owns the messages array and session, passing them down as props to `MessageList`/`MessageInput` rather than each component independently querying Supabase).
4. Move your existing CSS file over largely as-is — React doesn't require CSS-in-JS, plain stylesheets imported into `main.jsx` work fine at this stage.

**API surface:** None new — this is JSX structure and prop-passing.

**Verification:** The app renders the full layout with placeholder/fake data, styled correctly, before a single `useState` or Supabase call exists in the React version.

---

## Stage 3.3 — Auth State in React
**Goal:** `AuthGate` correctly shows `LoginForm` or `ChatLayout` based on real session state, and stays in sync.

**Tasks:**
1. In `AuthGate`, use `useState` to hold the current session (initially `null` or `undefined` to represent "not yet checked").
2. In a `useEffect` with an empty dependency array, call `client.auth.getSession()` on mount to set initial state, and call `client.auth.onAuthStateChange()` to keep it updated — remember to return the unsubscribe function from the effect's cleanup.
3. Rebuild `LoginForm` as a controlled component: input values live in `useState`, the submit handler calls `client.auth.signUp`/`signInWithPassword` exactly as before, just triggered from a React event handler instead of a vanilla listener.
4. Conditionally render `LoginForm` vs `ChatLayout` based on whether `session` is truthy.

**API surface:** Same Supabase Auth calls from Stage 2b.2/2b.3, now inside `useEffect`/`useState`.

**Search query if stuck:** `"react useEffect supabase auth onAuthStateChange cleanup unsubscribe"`

**Verification:** Logging in/out correctly swaps between `LoginForm` and `ChatLayout` with no full page reload, and refreshing mid-session preserves login state just like the vanilla version did.

---

## Stage 3.4 — Messages State & Realtime in React
**Goal:** `MessageList` renders live data from Supabase, updating in realtime, using React state instead of manual DOM manipulation.

**Tasks:**
1. In `ChatLayout` (or a dedicated custom hook if you want to get ahead of yourself — optional), hold messages in `useState([])`.
2. In a `useEffect`, call your fetch-history logic on mount and set it into state.
3. In the same or a separate `useEffect`, set up the realtime channel subscription exactly as in Stage 2a.4, but instead of manually creating DOM nodes in the callback, append the new row to your `messages` state array (`setMessages(prev => [...prev, payload.new])`).
4. Return the channel's unsubscribe function (`client.removeChannel(channel)` or the channel's own unsubscribe method) from the `useEffect` cleanup — this matters more in React than vanilla JS because component remounts would otherwise create duplicate subscriptions.
5. Pass `messages` down to `MessageList` as a prop; render bubbles via `.map()`, using each message's `id` as the React `key`.
6. Rebuild auto-scroll using `useRef` on the scroll container and a `useEffect` that runs `scrollTop = scrollHeight` whenever `messages` changes.

**API surface:**
- `useRef` for the scrollable container reference
- `client.removeChannel(channel)` for cleanup

**Verification:** Same two-window realtime test from Stage 2a.4 — should behave identically to the vanilla version, but now driven by React state with no manual DOM manipulation anywhere in the message rendering path.

---

## Stage 3.5 — Sending Messages in React
**Goal:** `MessageInput` sends messages exactly as before, now via React state and props/callbacks.

**Tasks:**
1. `MessageInput` holds its own draft text in local `useState`.
2. On submit (button click or Enter keydown handled via `onKeyDown` prop), call a `sendMessage` function passed down from `ChatLayout` as a prop, then clear local input state.
3. `ChatLayout`'s `sendMessage` function performs the same `client.from('messages').insert(...)` call as the vanilla version, including the `user_id` from session.
4. Resolve the same optimistic-UI-vs-realtime-only decision from Stage 2a.4 in this new structure — whichever way you chose before, replicate it, now via state updates instead of direct DOM appends.
5. Disable the send button while a send is in-flight, same as Stage 2a.5, using a `sending` boolean in state.

**API surface:** Same insert call as before, now invoked from a prop-passed callback.

**Verification:** Sending a message behaves identically to the vanilla + auth version from Stage 2b.4 — correct sender attribution, no duplicates, disabled button during send.

---

## Stage 3.6 — Cleanup and Parity Check
**Goal:** Confirm the React version has full feature parity with the vanilla version before moving on to new features.

**Tasks:**
1. Go through your Version 1 + 2a + 2b exit checklists again, this time testing the React app against every item.
2. Remove any leftover vanilla JS files, unused CSS classes from the old structure, and the CDN script tag if it's still in `index.html`.
3. Commit this as a clean milestone in git (`git commit -m "React refactor complete, feature parity with vanilla version"`) before starting Version 4 — this is a natural checkpoint to tag.

**API surface:** None — verification and cleanup only.

**Verification:** Every checklist item from Versions 1, 2a, and 2b passes in the React app with no regressions.

---

**Version 3 exit checklist:**
- [ ] Full component tree in place, each component has one clear job
- [ ] Auth, message history, and realtime all behave identically to the vanilla version
- [ ] Old vanilla files removed, milestone committed

---

# VERSION 4 — Rich Messaging Features

**Overall goal:** Multiple channels, image uploads, and typing indicators — the features that make it feel like a real product instead of a tech demo.
**Estimated time:** ~2 weeks, 20 hours.

## Stage 4.1 — Multiple Chat Channels
**Goal:** Users can switch between named channels (`#general`, `#tech`, `#random`), each with its own independent message history.

**Tasks:**
1. Add a `channels` table (`id`, `name`, `created_at`) and seed it with a few default rows via the dashboard.
2. Add a `channel_id` column to `messages`, referencing `channels.id`.
3. Build a channel list in `Sidebar`, fetched from the `channels` table, with a click handler that sets an `activeChannelId` in `ChatLayout` state.
4. Update your message fetch and insert logic to filter/scope by `activeChannelId` — your `.select()` call needs a `.eq('channel_id', activeChannelId)` filter, and your realtime subscription's filter object needs the same channel scoping so you're not receiving every channel's events in every view.
5. Update `MessageList` to re-fetch (or filter locally) whenever `activeChannelId` changes.

**API surface:**
- `.eq(column, value)` filter on `.select()`
- Realtime `on('postgres_changes', { event, schema, table, filter: 'channel_id=eq.X' }, ...)`

**Search query if stuck:** `"supabase realtime postgres_changes filter by column value"`

**Verification:** Switching channels shows a different, correctly-scoped message history in each, and sending in one channel doesn't leak into another channel's realtime updates.

---

## Stage 4.2 — Image Attachments
**Goal:** Users can attach and send an image, rendered inline in the chat bubble.

**Tasks:**
1. Create a Storage bucket in Supabase (e.g. `chat-images`), and decide on a folder structure (commonly `{user_id}/{timestamp}-{filename}`) to avoid collisions.
2. Add a file input (styled as an icon button, not a raw `<input type="file">`) to `MessageInput`.
3. On file selection, upload to Storage via `client.storage.from('chat-images').upload(path, file)`, then retrieve a public URL via `.getPublicUrl(path)` (or a signed URL if the bucket is private — decide based on whether images should be publicly linkable).
4. Add an `image_url` column (nullable) to `messages`. When sending an image message, insert the row with `image_url` set and `text` empty (or a caption if you support both).
5. Update bubble rendering to conditionally render an `<img>` when `image_url` is present, with a reasonable max-width/height so a huge image doesn't blow out the layout, and a loading placeholder while it loads.

**API surface:**
- `client.storage.from(bucket).upload(path, file)`
- `client.storage.from(bucket).getPublicUrl(path)`

**Search query if stuck:** `"supabase storage upload file getPublicUrl react example"`

**Verification:** Selecting an image, sending it, and seeing it render correctly-sized inline in the chat — and it appears via realtime in a second open window too, same as text messages.

---

## Stage 4.3 — Typing Indicators via Presence
**Goal:** "Alex is typing..." appears to other users in the same channel while someone is actively typing.

**Tasks:**
1. Read up on Supabase Presence as distinct from `postgres_changes` — Presence tracks ephemeral client state (who's online, what they're doing), not database rows.
2. Set up a presence channel scoped per-channel (reuse or parallel your existing channel-scoped realtime subscription from Stage 4.1).
3. On `MessageInput`'s `onChange`, track/untrack a "typing" presence state for the current user, debounced so you're not sending an update on every keystroke (track on first keystroke, untrack after ~2 seconds of no further input — a `setTimeout` reset on each keystroke works).
4. Subscribe to presence sync/join/leave events and derive a list of "who's currently typing, excluding me" to render as text under the message list.

**API surface:**
- `channel.track(stateObject)`
- `channel.untrack()`
- `channel.on('presence', { event: 'sync' }, callback)`

**Search query if stuck:** `"supabase presence typing indicator react example track untrack"`

**Verification:** Typing in one window shows "X is typing..." in a second window within about a second, and it disappears within ~2 seconds of stopping.

---

## Stage 4.4 — User Profiles
**Goal:** A `display_name` and optional avatar, replacing the raw email-prefix fallback from Stage 2b.4.

**Tasks:**
1. Add a `profiles` table (`id` uuid referencing `auth.users`, `display_name` text, `avatar_url` text nullable).
2. On sign-up (Stage 2b.2), after `signUp` succeeds, insert a corresponding row into `profiles` — decide whether to prompt for a display name at signup or default it to the email prefix and let users edit it later.
3. Build a simple profile-edit UI (accessible from the sidebar) letting users update their `display_name` and upload an avatar (same Storage pattern as Stage 4.2).
4. Update bubble rendering to join/look up the sender's `display_name` and `avatar_url` instead of showing a raw user id or email.

**API surface:** Same insert/update/storage patterns from earlier stages, applied to a new table.

**Verification:** Every message bubble shows a real display name and avatar (or a sensible default avatar) instead of any raw id or email fragment.

---

## Stage 4.5 — Notification Cues
**Goal:** Unread message counts and basic browser notifications for messages received while the tab isn't focused.

**Tasks:**
1. Track `document.hidden` / the `visibilitychange` event to know whether the tab is currently focused.
2. When a realtime message arrives for a channel that isn't the active one (or the tab isn't focused), increment an unread counter for that channel, shown as a badge in `Sidebar`.
3. Request Notification permission (`Notification.requestPermission()`) on first user interaction (not on page load — browsers deprioritize/block permission prompts that fire without a user gesture), and fire a browser notification for messages received while the tab is hidden.
4. Clear the unread badge for a channel when the user switches to it.

**API surface:**
- `document.visibilityState` / `document.addEventListener('visibilitychange', ...)`
- `Notification.requestPermission()`, `new Notification(title, options)`

**Verification:** With the tab unfocused or on a different channel, a new message increments the correct unread badge and (if permission granted) fires a system notification.

---

**Version 4 exit checklist:**
- [ ] Multiple channels work with correctly scoped history and realtime
- [ ] Image upload/send/render works end to end
- [ ] Typing indicators appear and clear correctly
- [ ] Real display names/avatars everywhere
- [ ] Unread badges and notifications work when tab is unfocused

---

# VERSION 5 — Security, Polish & Global Deployment

**Overall goal:** Lock down the database properly, polish rough edges, and ship it live with a real URL.
**Estimated time:** ~1–2 weeks, 15 hours.

## Stage 5.1 — Row Level Security Policies
**Goal:** Users can only read channels/messages they should have access to, and can only edit/delete their own messages — enforced at the database level, not just hidden in the UI.

**Tasks:**
1. Re-enable RLS on `messages`, `channels`, `profiles`, and your Storage bucket (all of which you may have left open during earlier versions).
2. Write a `SELECT` policy on `messages` allowing any authenticated user to read (adjust later if you add private channels).
3. Write an `INSERT` policy on `messages` requiring `auth.uid() = user_id` — this stops anyone from inserting a message pretending to be another user, even if they bypass your UI and call the API directly.
4. Write `UPDATE`/`DELETE` policies on `messages` requiring `auth.uid() = user_id`, so users can only edit/delete their own messages.
5. Write a similar `auth.uid() = id` policy pattern for `profiles` updates.
6. Test each policy by attempting a disallowed action directly via the Supabase JS client (e.g. try inserting a message with a different user's `user_id`) and confirming it's rejected, not just hidden by your UI.

**API surface:** Supabase SQL editor / dashboard policy builder — this is SQL/dashboard configuration, not JS.

**Search query if stuck:** `"supabase row level security policy auth.uid() insert update delete example"`

**Verification:** Attempting to insert/update/delete a row that violates a policy (tested via direct API call, not just UI) is rejected by Supabase with a clear permissions error.

---

## Stage 5.2 — Input Validation & Rate Limiting
**Goal:** The app doesn't break or get abused from malformed or spammy input.

**Tasks:**
1. Add server-side-equivalent constraints at the database level: `CHECK` constraints or column length limits on `messages.text` (e.g. max 2000 characters) so a client bypassing your UI can't insert absurdly long content.
2. Add basic client-side rate limiting on send (e.g. disable the send button for a short cooldown after rapid consecutive sends) to reduce accidental spam — note this is a UX nicety, not real protection, since it's trivially bypassed by anyone calling the API directly; real abuse prevention would need a server-side function, which is out of scope here but worth knowing the limitation of.
3. Sanitize any user-generated content that gets rendered as HTML (display names, message text if you ever render it as anything other than plain text) to prevent XSS — if you're using React's default JSX text rendering throughout, this is largely handled for you, but confirm you're not using `dangerouslySetInnerHTML` anywhere without sanitization.

**API surface:** SQL `CHECK` constraints; React's default escaping behavior.

**Verification:** Attempting to insert a message longer than your limit via direct API call is rejected by the database, and pasting `<script>alert(1)</script>` as message text renders as literal text, not executed script.

---

## Stage 5.3 — Environment & Secrets Hygiene
**Goal:** No credentials or sensitive config are exposed in your git history or client bundle in a way that matters.

**Tasks:**
1. Confirm `.env` is in `.gitignore` and was never committed — check `git log --all --full-history -- .env` to be sure.
2. Understand (and be able to explain) why the Supabase anon key is safe to expose client-side by design — it's meant to be public, with RLS doing the actual access control — versus the service_role key, which must never appear in any client-side code.
3. Double check nothing in your codebase references the service_role key.
4. Set up environment variables in your deployment platform (Vercel) matching your local `.env`, rather than hardcoding values anywhere.

**API surface:** None — configuration and git hygiene.

**Verification:** A fresh clone of your repo (without your local `.env`) fails to connect until environment variables are supplied — proving nothing sensitive is hardcoded.

---

## Stage 5.4 — Deploy to Vercel
**Goal:** A live, public URL running your production build.

**Tasks:**
1. Push your repo to GitHub if it isn't already there.
2. Connect the repo to Vercel, letting it auto-detect the Vite framework preset.
3. Add your environment variables (Stage 5.3) in the Vercel project settings.
4. Trigger a deploy and confirm the live URL works end-to-end: sign up, log in, send a message, see it in realtime from a second device/browser hitting the same live URL.
5. Set up automatic deploys on push to `main` so future changes go live without manual steps.

**API surface:** Vercel dashboard/CLI — no new code APIs.

**Verification:** The live production URL, opened on your phone and your laptop simultaneously, shows realtime message sync between the two, with a real account login on each.

---

## Stage 5.5 — Mobile & PWA Polish
**Goal:** The app can be installed like a native app on a phone home screen.

**Tasks:**
1. Add a `manifest.json` with app name, icons (multiple sizes), theme color, and `display: standalone`.
2. Link the manifest in your `index.html` head, along with appropriate `<meta name="theme-color">` and apple-touch-icon tags for iOS.
3. Test "Add to Home Screen" on an actual phone browser and confirm it launches without browser chrome, using your defined icon and theme color.
4. Do a final pass on mobile: test the full flow (login → send → receive → switch channel → upload image) on a real phone on real mobile data, not just a resized desktop browser window, since real touch/network conditions surface issues emulators miss.

**API surface:** `manifest.json` structure; standard PWA meta tags.

**Search query if stuck:** `"PWA manifest.json minimal example add to home screen"`

**Verification:** Installing the app to a phone home screen launches it standalone (no browser URL bar), and the full user flow works correctly over real mobile data.

---

**Version 5 exit checklist:**
- [ ] RLS policies tested and enforced at the database level, not just UI
- [ ] No secrets committed to git; service_role key never touches client code
- [ ] Live on a public Vercel URL with working realtime across real devices
- [ ] Installable as a PWA on mobile

---

# Appendix A — Full Skill Checklist by Version

| Version | Core Skills Gained |
|---|---|
| 1 | Flexbox, media queries, CSS custom properties |
| 2a | DOM manipulation, event handling, async/await, SQL basics, WebSocket-based realtime |
| 2b | Auth flows, session management, route guarding without a router |
| 3 | React component architecture, hooks (`useState`/`useEffect`/`useRef`), effect cleanup |
| 4 | Relational filtering, file storage/upload, Presence/ephemeral state, browser notification APIs |
| 5 | Row Level Security / database-level authorization, secrets hygiene, CI/CD deployment, PWA packaging |

# Appendix B — Where to Go From Here
Once Version 5 is done, natural next steps if you want to keep extending: message reactions/emoji, message editing with an "edited" indicator, read receipts (another Presence-adjacent pattern), infinite scroll/pagination for long channel histories instead of loading everything at once, and search across message history. None of these are required for a complete, deployable app — they're extensions once the core is solid.
