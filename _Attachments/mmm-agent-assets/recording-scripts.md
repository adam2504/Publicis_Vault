# MMM Agent, Recording Scripts

Record each scene as its own clip. Use macOS screen recording (`Cmd+Shift+5`, then "Record Selected Portion", or QuickTime, New Screen Recording). A few setup notes before you start.

- **Browser window**: resize to a clean 16:9-ish size (e.g. 1600x1000) before recording. Don't resize mid-clip.
- **Hide clutter**: close other tabs/apps, hide the bookmarks bar, turn off notifications (Do Not Disturb).
- **One take per scene.** Don't worry about mistakes, just start a new clip if you flub it. I'll cut the good parts.
- **Leave 2s of dead air** at the start and end of every clip (just sit still). Gives me clean handles to cut on.
- **Save each clip** with the scene number in the filename, e.g. `scene-01-fullpage-empty.mov`, into a folder you share with me after.
- **Client/data**: use Longchamp (`e68b8640`) unless a scene says otherwise, matches what I already captured for continuity. Real client data will appear on screen, that's expected. We'll decide later what, if anything, needs blurring for the public cut.
- **Pace**: move deliberately, not fast. Pause about 1s after every click before your next action. Motion design editing needs clean beats to cut on, not a rushed click-through.

Total target: about 8 short clips, each 10 to 25s raw (I'll trim tighter in the edit). Don't aim for a polished performance, just clean, deliberate actions.

---

## Scene 1, Full-page assistant, empty state
**Goal:** set up the AI Assistant page calmly, showing the layout and the 3 suggestion chips.

1. Navigate to `https://connectedhub.publicismedia.com/e68b8640/marketing-mix-modeling/chatbot`
2. Wait for it to fully load. Sit still 2s.
3. Slowly move your mouse over the three suggestion buttons (don't click), pausing briefly on each.
4. Sit still 2s. Stop recording.

## Scene 2, Asking a question, live status streaming
**Goal:** capture the SSE status messages changing in real time. This is the money shot for the transparent, step-by-step architecture.

1. Same page as Scene 1 (fresh page reload so it's a clean "New conversation").
2. Click the suggestion "Which media channel has the best ROAS?"
3. Do not touch anything, just let it run. Let the camera sit on the status bubble as it changes (you'll see it cycle through phrases like "Reading the data model...", "Discovering your KPIs and channels...", "Verifying the results...").
4. Keep recording until the final answer appears and finishes rendering.
5. Sit still 2s. Stop recording.

*(This one clip may run 15 to 25s since the pipeline takes a few seconds, that's fine, real duration is part of the point.)*

## Scene 3, Live screen-scope awareness (hero moment)
**Goal:** prove the agent reads the KPI and period currently on screen, not a hardcoded default.

1. Go to the Results page first: `https://connectedhub.publicismedia.com/e68b8640/marketing-mix-modeling/results`
2. In the left sidebar, change the Period to a different range than default (click the period fields, pick a visibly different week range).
3. Open the floating AI Assistant bubble (bottom-right).
4. Type a question that does not mention any KPI or period: "What's my overall ROAS?" and press Enter.
5. Let it run fully. When the answer appears, pause 3s on it, it should cite the exact KPI and time period you just set. This is the key proof point, hold the shot.
6. Stop recording.

## Scene 4, Follow-up question, conversation memory
**Goal:** show it remembers context across turns.

1. Continue directly from Scene 3 (same page, same conversation, don't reset).
2. Type a follow-up that only makes sense with memory: "And which channel drove that the most?"
3. Let it fully answer. Pause 2s on the final answer.
4. Stop recording.

## Scene 5, Bubble persists across tabs
**Goal:** show the floating widget stays open while navigating the module.

1. With the bubble still open (continue from Scene 4, or reopen it on Results if you cut), click the Model link in the left sidebar (not a full page reload, just the sidebar nav link).
2. Pause 2s once the Model page loads with the bubble still floating open in the corner.
3. Click Simulation in the sidebar.
4. Pause 2s again with the bubble still open.
5. Stop recording.

## Scene 6, Chart-reading help
**Goal:** show the assistant explaining a specific chart, tied to what's on screen.

1. Fresh page: `https://connectedhub.publicismedia.com/e68b8640/marketing-mix-modeling/model`
2. Open the bubble, click "How do I read the saturation curves?" suggestion (or type it if not shown).
3. Let it answer fully. Pause 2s at the end.
4. Stop recording.

## Scene 7, Multi-client switch
**Goal:** prove this isn't a single-client demo. Same agent, different client, different KPI.

1. Navigate to `https://connectedhub.publicismedia.com/kw3oVgiDE/marketing-mix-modeling/chatbot` (Opel DE, notice the KPI is "Leads volume" and there's a Granularity toggle, unlike Longchamp).
2. Sit still 2s so the different sidebar (different KPI, different market "Germany") is clearly visible.
3. Click "Which media channel has the best ROAS?" suggestion.
4. Let it answer fully. Pause 2s.
5. Stop recording.

## Scene 8, Wide establishing shot (optional but nice)
**Goal:** a slow establishing pan across a full dashboard with the module nav visible, for use as an intro or outro backdrop.

1. Go to `https://connectedhub.publicismedia.com/e68b8640/marketing-mix-modeling/results`
2. Don't touch anything. Just let it sit for 5s, I'll speed-ramp or use this as a static backdrop behind title text.
3. Stop recording.

---

## After recording

Drop all `.mov` files into one folder (Downloads is fine) and tell me the path. I'll pull them into the Remotion project, cut them down, add the pan, zoom, highlight, and caption motion design on top, and render the final 16:9 MP4.
