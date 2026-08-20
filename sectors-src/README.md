# 11Sight · Booking Agent Provisioning Page

Self-serve provisioning demo for the AI Booking Agent. This is the page that opens
inside the "Try agent" popup on the AI Booking Agents landing page, and it also
works standalone. Live: https://provisioning-demo.vercel.app/sectors/

## Files

| File | What it is |
|---|---|
| `provisioning-v2-sectors.html` | The page. All CSS and JS live inside this file (one `<style>` block, one `<script>` block). It links `inter.css` for fonts. |
| `inter.css` | Inter font faces (400/500/600/700), base64-embedded. Keep it next to the HTML. |
| `../sectors/index.html` | The same page as a single self-contained file (fonts inlined). This is what production serves. |

Open `provisioning-v2-sectors.html` in a browser and everything runs; no build step,
no dependencies.

## What the page contains, in flow order

1. **Password gate** — the whole demo sits behind a password (`11future`). It is
   client-side only, meant to keep casual visitors out, not real security. A correct
   entry is remembered for the browser session (`sessionStorage['prov-gate']`).
   Visitors without a password can book a meeting from the gate (embedded HubSpot
   calendar). After the gate the page opens the dental setup directly; `#studio`
   deep-links into the studio. The landing popup passes the selected agent in the
   iframe URL as `?agent=<id>` (e.g. `?agent=dental`; a bare `#dental` hash works
   as a fallback) — see `state.agentId` below.
2. **Agent setup** (view 1) — basic form: business name, agent name, timezone,
   confirmations (email is required, validated), greeting, languages. The left panel
   is a live intro: SERENA wordmark, particle-sphere canvas, photo background, and a
   greeting bubble that mirrors the form as you type.
3. **Agent studio** (view 2) — "Advanced Customization": 8 step panels (Appointments,
   Rules, Knowledgebase, Transfers, Call Opening/Closing, Persona, Identity). Every
   change updates the live demo rail on the right. Picking a Persona restyles the
   demo (orb palette + stage colour).
4. **Live demo rail** — the test agent: canvas particle orb, "Talk to Agent" button,
   status chips. A test call animates the stage; the real audio is stubbed.

## Integration points (search for `DEVELOPER HANDOFF` in the script)

The UI is done; the backend is stubbed at exactly these seams:

- **`state.agentId`** — which agent this page instance represents. Read once at boot
  from the iframe URL (`?agent=<id>`, with a bare `#<id>` hash as fallback; defaults
  to `dental`), mirrored to `<body data-agent="…">`, and included in every
  `pushConfigToBackend(state)` payload.
- **`state`** — single source of truth for the whole configuration. Inputs are wired
  via `data-bind` paths; every change funnels through `queueSync()` → `syncDemo()`
  (updates the demo rail) → `pushConfigToBackend(state)` **(stub: send the config to
  your provisioning API here)**.
- **`startCall()` / `endCall()`** — the "Talk to Agent" button. Open/close the real
  voice session here. The entire UI is driven by the `.is-live` class on `#sd-stage`;
  you do not need to touch any animation code.
- **`launch-btn`** — first-time provisioning submit (setup → studio transition).
- **Gate password** — `GATE_PASS` constant in the script. Replace with a real check
  if it ever needs to be more than a demo gate.

## Conventions to keep

- New fields: use the same `data-bind` path pattern and let changes flow through
  `queueSync()`. New demo indicators: add a `data-demo` attribute and update it from
  `syncDemo()`.
- Seeded list items are protected (`seedLock` / `isSeed`): stock entries cannot be
  deleted from the UI. Newly added items are drafts until saved (`commit-item`).
- The canvas painters (sphere, light bodies, photo blend) are design-approved.
  Integration never requires editing them.
