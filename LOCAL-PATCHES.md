# Local patches

This fork carries patches that run as a local build on top of the official
Nimbalyst release (currently **v0.79.1**, macOS arm64). They are applied to the
minified app bundle by a patch script, each with its own test, and the app is
re-signed ad hoc. Each entry says what the patch does, why it exists and
whether there is an upstream PR or issue. Upstream branch PRs are cut from
`nimbalyst/main`; this `main` only adds this file.

| Patch | What it does | Why | Upstream |
|---|---|---|---|
| plan/Codex (`plan-agent`) | Plan/Agent toggle for Codex sessions: read-only plan mode, plan save, plan completion dialog with explicit approval and a fresh execution session. | Codex sessions had no plan mode. | Issue [#1564](https://github.com/nimbalyst/nimbalyst/issues/1564) |
| plandialoog (+ 24 h) | A user `PreToolUse` hook on `ExitPlanMode` gates the plan dialog, and gets `timeout: 86400`. | The dialog rendered before the hook ran, and the CLI killed the hook after 600 s, so the dialog vanished after 10 minutes. | Issue [#1561](https://github.com/nimbalyst/nimbalyst/issues/1561) |
| widgets (`vraagwidget`) | A question widget whose answer failed shows "Question failed / Not answered" instead of "Questions Answered". | A failed AskUserQuestion looked answered. | none |
| compact | `/compact` works in Codex sessions through the real Compact action. | Codex sessions ignored `/compact`. | Issue [#1745](https://github.com/nimbalyst/nimbalyst/issues/1745) |
| overlay (`browser-overlay`) | Hidden browser surfaces report no bounds; main intersects bounds instead of clamping them onto the window. | Off-screen browser tabs (at -9999 px) were moved to the top-left corner over the app. | Issue [#1547](https://github.com/nimbalyst/nimbalyst/issues/1547) (closed; dff066b covers tab switches only) |
| bestandenpaneel (files panel) | The "All Uncommitted Files" sidebar starts closed everywhere and stays closed after restart; only a click opens it. | It reopened on every session and restart. | Issue [#1746](https://github.com/nimbalyst/nimbalyst/issues/1746) |
| quit-backup | Cmd+Q skips the database backup when the last successful one is younger than 24 h. | The 3.4 GB quit backup aborted after 5 s and crashed the app. | none |
| spawn-cap (`agents-50`) | A session may run 50 spawned sessions at once (upstream 4) and keep 200 unarchived children (upstream 50). | Swarms of 20–50 read-only agents. | Issue [#1747](https://github.com/nimbalyst/nimbalyst/issues/1747) |
| stop-final | Stop also fails the session's pending queued prompts, cancels its wakeup and stops parent notifications. | After Stop the next queued prompt started at once and the parent was woken. | Issue [#1744](https://github.com/nimbalyst/nimbalyst/issues/1744) |
| browser-focus | Agent browser tools type and click through CDP with focus emulation. | `sendInputEvent` needs OS window focus, so headless sessions and background tabs got no input. | PR [#1727](https://github.com/nimbalyst/nimbalyst/pull/1727), issue [#1726](https://github.com/nimbalyst/nimbalyst/issues/1726) |
| browser-tabs | `browser.open_session` opens a background tab "agent · site" without switching; agent tabs never opened are cleaned up after 15 min. | Agent browsing was invisible or stole the active tab. | Issue [#1728](https://github.com/nimbalyst/nimbalyst/issues/1728) |
| browser-login (1–6) | Browser logins persist on disk (`persist:preview`); `open_session` with `foreground: true` for a login or captcha, without the screen jumping to it; no false "File updates delayed" banner on browser tabs; a `sessionId` the agent picks itself ("ebay-dev") becomes an alias for its tab, resolved by every `browser.*` tool; a `.browser.json` tab that was never shown is parked at 1280×900 before any browser tool; the `open_session` description tells agents to log in themselves (stored credentials, codes from mail, consent buttons) and to use `foreground` only for captcha, passkey or phone approval. | Logins were lost on restart and the user could not find the tab that needed them; a self-chosen `sessionId` silently opened an invisible headless session and ignored `foreground`; a never-shown `.browser.json` tab had a 0×0 page, so agents asked the user to open it; agents asked the user to log in and click "Agree" although they had the credentials. | Issue [#1728](https://github.com/nimbalyst/nimbalyst/issues/1728) (comment) |
| agent-sessietab (agent session tab) | An agent's browser tab opens in the tab bar of that agent's own session (Agent mode), not between the Files tabs. | Agent tabs cluttered the file tabs and were hard to match to their session. Part of browser-login-2. | none |
| stall-sleep | The stream-stall watchdog counts only awake time; a tick that arrives more than 60 s late (sleep, frozen process) is not counted as silence. | A single `setTimeout(600 s)` counted Mac sleep and aborted a healthy turn one second after wake. | PR [#1749](https://github.com/nimbalyst/nimbalyst/pull/1749) |
| media-binair | `read-file-content` treats audio/video and webp/avif/heic as binary; chardet samples the first 64 KB. | An open mp4 tab was decoded as text on every window focus; chardet over the whole file froze the main process 6–8 s every 30–60 s. | PR [#1790](https://github.com/nimbalyst/nimbalyst/pull/1790) |

Patches whose upstream fix ships are dropped at the next version update.
