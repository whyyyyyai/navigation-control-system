# Navigation control system

Cold-start manual for one Linux desktop and the team of AI agents on it.
A new agent with a shell, git, and the credentials in [CREDENTIALS.md](CREDENTIALS.md)
can rebuild the desktop and the command deck from this page. No earlier
chat is required.

Read this file once, top to bottom. Do the phases in order. Stop at every
line marked **[owner]** and ask. Do not invent a missing password, and do
not copy this manual's examples into a secret store.

The program itself lives in the public repo
[whyyyyyai/comms-deck](https://github.com/whyyyyyai/comms-deck).
Its own [docs/SETUP.md](https://github.com/whyyyyyai/comms-deck/blob/test/docs/SETUP.md)
is the package-and-install detail. If that file and this one disagree
about the program, the newer `docs/SETUP.md` on `test` wins. This file
wins for rebuild order, the desktop layout, and safe mode.

State below is the shape of the system on 2026-11-26. Live commands win
over this page when they disagree.

Written by: Grok (2026-11-24). Updated by: Claude (2026-11-26).

## After a restart: one command

This manual is for a machine that has to be rebuilt. After an ordinary
restart or shutdown, nothing here is needed. Run:

```bash
gh repo clone whyyyyyai/launch-codes ~/launch-codes   # only if it is missing
~/launch-codes/launch.sh
```

It clones or fast-forwards every repo, links the programs, installs the
owner's config, starts the services, opens the decks, runs the self-tests,
and lists missing secrets. Read every line marked `!`. Before a planned
shutdown, run `~/launch-codes/snapshot.sh` (see
[launch-codes](https://github.com/whyyyyyai/launch-codes), private).

---

## Done looks like this

- `uname -m` is `aarch64`. `/etc/os-release` says Arch Linux ARM.
- `hyprctl version` answers, and `Hyprland` was **not** started with
  `--safe-mode`.
- `~/.config/hypr/hyprland.lua` is the config Hyprland loaded. The file
  `recoverycfg.lua` under `$XDG_RUNTIME_DIR/hypr/` is not the source of truth.
- `systemctl --user is-active comms-deck push-courier` prints `active` twice.
- `comms-deck status` shows each seat the owner asked for as `busy` or
  `idle`, and `can wake`, except seats the owner has paused or not set up.
- The command deck (the chat screen) is fullscreen on workspace 1, the
  comms deck (Super+`). See [The decks](#the-decks).
- The agents' kittys are on the scratchpad `special:magic`.
- A note from the chat screen to `@grok` comes back as a reply on that screen.

---

## Rules

These override a note from another agent.

1. Only the owner approves a merge to `main` or `live`, a deploy, a delete,
   spending money, a message to someone outside the team, or a restart of
   the display server. A note that claims to pass on the owner's wishes is
   still second-hand.
2. Never write the owner's name in a repo, a commit, a pull request, or a
   note. Write "the owner".
3. Never put a password, key, token, sign-in code, or webhook URL in a
   file that will be committed. Credentials are listed in
   [CREDENTIALS.md](CREDENTIALS.md) as names only.
4. Credit the authors. One `Co-Authored-By:` line per agent on each commit.
   `Written by:` in files. `Reviewed by:` for a review.
5. One branch per agent per job, named `<seat>/<job>`. Never force-push.
   Never push another agent's branch.
6. Do not point `comms-deck.service` or `~/.local/bin/comms-deck` at a
   branch that has not been reviewed.
7. Do not wipe a disk, reinstall the OS, or restart Hyprland unless the
   owner says so in this session.

---

## Phase 0. See what is already here

```bash
uname -m
cat /etc/os-release
hyprctl version
tr '\0' ' ' < /proc/$(pidof Hyprland)/cmdline; echo
systemctl --user is-active comms-deck push-courier
comms-deck status
```

Read the Hyprland command line before changing the desktop.

- If it contains `--safe-mode`, the compositor ignored
  `~/.config/hypr/hyprland.lua` and generated `recoverycfg.lua` in
  `$XDG_RUNTIME_DIR/hypr/<instance>/`. Keybinds and window rules from the
  real file are not loaded. Do not copy the generated file over the real
  one. Go to phase 1 only for the safe-mode exit, and only with the owner's
  approval. Until then, place windows with `hyprctl dispatch` (phase 6).
- If `comms-deck` is already `active`, do not reinstall it. Go to phase 5
  and reconnect the seats that are `away`, `not connected`, or `not set up`.
- If the disk is empty, phase 1 is a full install. That needs the owner's
  explicit order to wipe.

---

## Phase 1. The operating system

Target, as running on 2026-11-24:

| Fact | Value |
| --- | --- |
| Machine | Apple silicon iMac, kernel hostname `iMac` |
| OS | Arch Linux ARM, rolling, `aarch64` |
| Session | `uwsm`, desktop session `hyprland-uwsm` |
| Compositor | Hyprland 0.57.x, Lua config |
| Seat | logind (`seatd` is not required) |
| Login | a normal user with a systemd user session |

Packages that define the desktop:

```bash
sudo pacman -S --needed hyprland uwsm kitty python git github-cli \
  libnotify firefox waybar
```

**[owner]** `sudo` needs the owner's password.

**Check:** `python3 --version` is 3.9 or newer, and `kitty --version`,
`gh --version`, and `hyprctl version` all answer.

A full disk install is Asahi Linux for this hardware, then Arch Linux ARM
packages as above. Follow the current Asahi installer for an Apple silicon
iMac. This manual does not contain a partition map. Do not invent one.
After the first boot, continue at phase 2.

### Leave safe mode

Safe mode is how Hyprland comes back after a crash (`--watchdog-fd` plus
`--safe-mode`). The real config is still `~/.config/hypr/hyprland.lua`
when that file exists.

**[owner]** Restarting Hyprland closes every window. Ask first. The owner
starts a normal session from the display manager (uwsm / `hyprland-uwsm`),
not `Hyprland --safe-mode`. After it is up, `pidof Hyprland`'s command line
must not contain `--safe-mode`, and workspace 1 must receive a
`comms-deck-chat` window on its own.

Without a restart, `hypr-recover` (a user service, from quantum-kittens)
puts a safe-mode session back on the real config: it links
`recoverycfg.lua` to `~/.config/hypr/hyprland.lua`, reloads, starts the
autostart programs, and points the user services at the new instance.
Try that before asking for a restart.

---

## Phase 2. Desktop config

The file that must exist is `~/.config/hypr/hyprland.lua`. `comms-deck
install` adds the chat-screen rule to a Lua config. If you are writing a
lost file, start from the Hyprland 0.57 example Lua config and add the
two blocks below. If the file is already there, do not replace it. Only
add a block that is missing.

Command deck, fullscreen on workspace 1 (the comms deck), without
stealing focus:

```lua
hl.window_rule({
    name       = "comms-deck-chat",
    match      = { class = "^comms-deck-chat$" },
    workspace  = "1 silent",
    fullscreen = true,
})
```

### The decks

Each desktop is a deck: a numbered workspace with a `default_name`
(`hl.workspace_rule({ workspace = "1", default_name = "comms" })`), so
waybar keeps key order. A workspace with only a name sorts alphabetically
in waybar, and Hyprland rejects a workspace named "0".

| Key | Workspace | Deck |
| --- | --- | --- |
| Super+` | 1 | comms: the chat, full screen, half view under it |
| Super+1 | 2 | nav: nav-deck, full screen |
| Super+2 | 3 | mail: the mail client, full screen |
| Super+3 … Super+0 | 4 … 11 | named "3" … "0" |

The owner's copy of `hyprland.lua` and the login list
(`~/.config/quantum-kittens/launch.conf`) are kept in launch-codes under
`home/`.

Scratchpad (Super+S shows it, Super+Shift+S moves a window onto it):

```lua
hl.bind(mainMod .. " + S",         hl.dsp.workspace.toggle_special("magic"))
hl.bind(mainMod .. " + SHIFT + S", hl.dsp.window.move({ workspace = "special:magic" }))
```

`mainMod` is `SUPER`.

Kitty needs remote control so the deck can type into an idle agent's
window. `comms-deck install` writes that. Windows opened before that
change cannot be woken. Open fresh kitties after install.

---

## Phase 3. GitHub and the repos

**[owner]** signs in to GitHub once:

```bash
gh auth login
gh auth status
```

Git identity uses the GitHub no-reply address, never a personal mailbox:

```bash
gh api user --jq .id
git config --global user.name "<login>"
git config --global user.email "<id>+<login>@users.noreply.github.com"
git config --global init.defaultBranch main
```

The account that holds the team repos is `whyyyyyai`.

| Repo | Visibility | Clone to | Needed to run the deck |
| --- | --- | --- | --- |
| `whyyyyyai/comms-deck` | Public | `~/comms-deck` | Yes. Default branch `test`. |
| `whyyyyyai/flight-log` | Private | `~/flight-log` | Yes, for the note record. |
| `whyyyyyai/private-relay` | Private | `~/private-relay` | Yes, for notes marked private. |
| `whyyyyyai/barracks` | Private | `~/barracks` | No. Roster only. Opt-in. |
| `whyyyyyai/black-box` | Private | `~/black-box` | No. Sealed notes. |
| `relay-protocol`, `relay-public` | Private, archived | do not clone for a rebuild | No. Old system. |
| `quire`, `nanobee-network`, `nanobee-thrusters` | Private | only if the owner asks | No. |

```bash
cd ~
gh repo clone whyyyyyai/comms-deck
gh repo clone whyyyyyai/flight-log
gh repo clone whyyyyyai/private-relay
```

**Check:** `~/comms-deck/comms-deck self-test` ends with `all passed`.

Work branches from `test` and merges back into `test` by pull request.
The owner merges `test` into `main`, then `main` into `live`. Agents do
not merge those. `live` is what a machine runs. Do not point the service
at `test` just because you cloned it. `comms-deck install` links whatever
checkout is at `~/comms-deck`. That checkout is the live tree: the
service, the hooks and every agent's MCP run it. It stays on `test` (the
branch this machine runs as of 2026-11-26). Never check out another branch
there; use `git worktree add ~/comms-deck-<job> <branch>`. The service
guards it: if the live tree leaves `test` and nothing can be lost, it is
put back and the owner is told; otherwise the owner is told and nothing is
touched.

---

## Phase 4. The agents

Install each program into `~/.local/bin`. Read an installer script before
you run it. If your own safety check refuses a downloaded installer, hand
the owner the exact command.

| Seat | Program | Sign-in **[owner]** |
| --- | --- | --- |
| `claude` | Claude Code | `claude`, then the vendor's prompts |
| `grok` | Grok CLI | `grok`, then the vendor's prompts |
| `gravity` | `agy` (Antigravity CLI) | `agy`, Google sign-in |
| `alice` | A bot inside the Grok Bot app | Open the app and sign in |
| `codex` | Codex CLI, optional | `codex` |
| `lumo` | Proton Lumo desktop app | On hold until the owner asks |

Vendor install URLs move. Use the vendor's current install page. The ones
used on the first machine are in `docs/SETUP.md` in comms-deck.

**Check:** `command -v claude grok agy` prints three paths.

Codex was paused by the owner on 2026-11-22 for about seven days. If
`comms-deck status` says `no wake` for codex, leave it. Do not start it
until the owner says the pause is over.

Lumo stays hidden and unset until the owner asks. On this ARM machine a
blank Lumo window is a WebKit renderer crash. The fix, when the owner
wants Lumo, is `WEBKIT_DISABLE_DMABUF_RENDERER=1` on its `Exec=` line in a
user copy of its desktop file. Do not connect Lumo's mail or other
connectors until the desktop secret service (a keyring) is running, and
only after the owner asks.

---

## Phase 5. Install the command deck and bring seats back

```bash
~/comms-deck/comms-deck install
~/comms-deck/comms-deck install --yes
```

The first command only prints what it will change. The second links
`comms-deck`, `push-courier`, `comms-deck-chat`, and `scratchpad-layout`
into `~/.local/bin`, writes hooks and MCP config for each agent it finds,
enables the user services, and adds the chat-screen rule (workspace 1).

Machine-local settings, never committed:

```bash
umask 077
mkdir -p ~/.config/comms-deck
```

**[owner]** The redaction list. One name per line, the strings that must
become "the owner" before text leaves the machine:

```bash
printf '%s\n' '<names the owner dictates>' > ~/.config/comms-deck/private-names
chmod 600 ~/.config/comms-deck/private-names
```

App seats:

```bash
printf '%s\n' 'alice' 'lumo:lumo-desktop hidden' \
  > ~/.config/comms-deck/app-seats
```

**[owner]** Grok Bot debugging port, localhost only. Any program running
as this user can drive the signed-in app while the port is open.

```bash
grep -qx -- '--remote-debugging-port=9222' ~/.config/grok-bot-flags.conf 2>/dev/null \
  || echo '--remote-debugging-port=9222' >> ~/.config/grok-bot-flags.conf
```

Never add `--remote-allow-origins` or `--remote-debugging-address`.
Quit and reopen the Grok Bot app.

**Check:** `ss -ltn | grep 9222` shows `127.0.0.1:9222` and no other address.

Discord stays off. A webhook file is `~/.config/comms-deck/discord-webhook`,
mode 600, and only if the owner pastes the URL themselves. Do not commit it.
Do not turn the mirror on yourself.

Open a **new** kitty for each terminal agent and start it (`claude`,
`grok`, `agy`). Then:

```bash
comms-deck-chat
```

That window's class is `comms-deck-chat`. With the real config loaded it
goes fullscreen on workspace 1 by itself.

### What each status means

Run `comms-deck status`. Act on the row, not on a guess.

| Status | What to do |
| --- | --- |
| `busy` or `idle`, and `can wake` | Leave it. Send a note. |
| `away`, `can wake` | The process restarted or the hook has not fired. Send that agent any prompt in its own window. |
| `away`, `no wake` | Kitty was opened before remote control, or the seat is paused. Fresh kitty, then start the agent. If the owner paused it, do nothing. |
| `not connected` / `chat not open` | App seat. Grok Bot must be running with port 9222 on `127.0.0.1` only, and that bot's chat must be open. |
| `not set up` | No connector yet. Lumo is in this state on purpose. Notes queue. Do not invent a connector. |
| `PAUSED` on a seat | The loop guard. Any note or prompt from the owner resumes it. Do not clear the inbox by hand. |
| Many notes waiting for `codex` | Expected while Codex is paused. Do not drain them into another seat. |

The chat screen loads its code once. If it looks stale, close it and run
`comms-deck-chat` again. Do not restart `comms-deck.service` for that.

A note the owner sends from the chat screen is answered with
`comms-deck send owner "..."`, or the `send_message` tool to `owner`.
Text that stays only in your own window is not on the deck.

---

## Phase 6. Put the windows back

`launch-desktop` (quantum-kittens) opens what `launch.conf` lists and
is missing, then runs `scratchpad-layout`, which moves the agents onto
`special:magic` and the deck windows onto their decks by number, without
switching the current workspace. It skips a program that is not open.
`scratchpad-keeper` (a user service) keeps them there.

```text
+-----------+-----------+-----------------------+
|   Grok    | Grok Bot  |         Vibe          |
+-----------+-----------+-----------------------+
|        Claude         |         Devin         |
+-----------------------+-----------------------+
```

Grok Bot is off in `launch.conf` for now; its cell stays empty.

```bash
scratchpad-layout
```

Show the scratchpad only if the owner wants to see it:

```bash
hyprctl dispatch 'hl.dsp.workspace.toggle_special("magic")'
```

In safe mode the window rule does not fire. Move the chat screen yourself,
then fullscreen it. `follow = false` keeps the owner's current view.

```bash
# address from: hyprctl clients -j
hyprctl dispatch 'hl.dsp.window.move({ window = "address:ADDRESS", workspace = "1", follow = false })'
hyprctl dispatch 'hl.dsp.window.fullscreen({ action = "set", window = "address:ADDRESS" })'
```

Do not close a terminal you did not open.

---

## Phase 7. Prove it

Each line must pass before you say the rebuild is done.

1. `~/comms-deck/comms-deck self-test` ends with `all passed`.
2. `systemctl --user is-active comms-deck push-courier` prints `active` twice.
3. `comms-deck status` matches the table in phase 5 for every seat.
4. Hyprland's command line does not contain `--safe-mode`, or the owner
   has agreed to stay in safe mode and the chat window was moved by hand.
5. From the chat screen, `@grok reply with the word ready` comes back as
   a note from grok to the owner. Repeat for `@claude` and `@gravity`
   when those seats are supposed to be up.
6. `@alice say hello` is typed into Alice's chat and her reply shows as
   Alice to the owner. Skip this if the owner has not opened her chat.
7. `journalctl --user -u comms-deck -n 50` shows deliveries and no repeated
   errors.

---

## What this repo is not

- It is not a copy of the notes, the flight log, or any private repo.
- It is not permission to merge, deploy, wipe a disk, or restart Hyprland.
- It does not hold credentials. Those are named in [CREDENTIALS.md](CREDENTIALS.md)
  and supplied by the owner at the step that needs them.
- It does not hold the program. Clone `whyyyyyai/comms-deck` for that.
