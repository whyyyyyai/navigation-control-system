# Credentials a rebuild needs

Names only. Values are never written here, in a commit, or in a note.
The owner supplies each one at the step that needs it, by signing in
themselves or by pasting into a mode-600 file that is not in git.

Written by: Grok (2026-11-24).

| What | Who holds it | Where it goes | Phase |
| --- | --- | --- | --- |
| Linux sudo password | owner | typed at the sudo prompt, not stored by an agent | 1 |
| GitHub login for `whyyyyyai` | owner | `gh auth login` on this machine | 3 |
| Access to the private repos (`flight-log`, `private-relay`, and any other the owner names) | that same GitHub login | clone only. Do not mirror them into a public repo | 3 |
| Claude sign-in | owner | the `claude` prompt | 4 |
| Grok and Grok Bot sign-in | owner | the `grok` prompt and the Grok Bot app | 4 |
| Google sign-in for Antigravity (`agy`) | owner | the `agy` prompt | 4 |
| Codex sign-in | owner | only after the owner ends the pause | 4 |
| Proton account for Lumo | owner | only after the owner asks for Lumo | 4 |
| Strings to redact | owner | `~/.config/comms-deck/private-names`, mode 600 | 5 |
| Approval for the Grok Bot debugging port | owner | one line, `--remote-debugging-port=9222`, localhost only | 5 |
| Discord webhook | owner | `~/.config/comms-deck/discord-webhook`, mode 600, and only if the owner wants the mirror | 5 |
| Approval to leave Hyprland safe mode | owner | a normal session start. It closes every window | 1 |
| Approval to wipe a disk | owner | said in this session, before any partition tool runs | 1 |

If a value is missing, stop and ask the owner. Do not look for it in an
old chat, a browser cookie, or another agent's notes.
