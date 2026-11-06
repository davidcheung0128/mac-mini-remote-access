# HG7 Mac mini remote access

How a coworker connects to the HG7 Mac mini from their own Mac with Tailscale and macOS Screen Sharing (VNC), and how to fix it when it times out. **Everything you need is on this page.** The other files have extra detail.

This repository is public. Every address, account and email here is a placeholder. Get the real values privately from the host administrator, and never put passwords, keys or invitation links here.

## Connection details

| Item | Value |
| --- | --- |
| Machine | HG7 Mac Mini / `hg7-macmini` |
| HG7 Tailscale address | `HG7_TAILSCALE_IP` |
| Coworker's account on HG7 | `coworkeraccount` (a Standard account, not an admin one) |
| Email the Tailscale share was sent to | `COWORKER_INVITED_EMAIL` |
| Coworker's own Mac's Tailscale address | `COWORKER_TAILSCALE_IP` |
| Screen Sharing port | TCP 5900 (SSH: TCP 22) |

## How to connect (when everything is set up)

1. Turn on **Tailscale** on your Mac (menu-bar icon → **Connected**).
2. Finder → **Command+K** → `vnc://HG7_TAILSCALE_IP` → Connect.
3. Log in with `coworkeraccount` and your **HG7** password (not your Tailscale or own-Mac password).
4. If asked, choose **Log in as yourself**.
5. When finished, close the window or log out. **Never Restart or Shut Down HG7.** It runs shared work systems.

## ⭐ It times out: try these first

### 1. Host administrator: allow the coworker to use Screen Sharing

Without this, HG7 refuses the login even when the network works. On HG7, with an admin account:

1. **System Settings → General → Sharing**.
2. Click **ⓘ** next to **Screen Sharing**.
3. Under **Allow access for: Only these users**, click **+**, choose the coworker's account, then **Select → Done**.

Or in Terminal on HG7 (it asks for the admin password):

```bash
sudo dseditgroup -o edit -a coworkeraccount -t user com.apple.access_screensharing
```

Check it worked. It should print `user is a member of the group`:

```bash
dsmemberutil checkmembership -U coworkeraccount -G com.apple.access_screensharing
```

### 2. Coworker: be signed in to Tailscale with the invited email

"Connected" in Tailscale does **not** mean you can reach HG7. HG7 is shared with one specific account. If your Mac is signed in with a different email, or is a re-registered device, HG7 can't see it.

Paste into Terminal on **your own Mac**:

```bash
TS=/Applications/Tailscale.app/Contents/MacOS/Tailscale; $TS status | head -3; $TS switch --list
```

- The first line is your own Mac. It shows your address and your account.
- In the account list, the one marked `*` must be `COWORKER_INVITED_EMAIL`.
- **hg7-macmini** should appear in the device list.

**If the account is wrong or hg7-macmini is missing**, log out and back in:

```bash
TS=/Applications/Tailscale.app/Contents/MacOS/Tailscale; $TS logout; $TS up
```

A browser login page opens. Sign in with `COWORKER_INVITED_EMAIL`, using the same method the invitation used (e.g. **Sign in with Google** for a Gmail address). If hg7-macmini still isn't listed, open the invitation email and click **Accept** while signed in with that email, or ask the host administrator to resend the share.

### 3. Coworker: quit other VPN and proxy apps, then restart

Other VPNs and proxies (NordVPN, Clash/Surge, company VPNs) can take over HG7's address, even when switched off.

1. Quit them completely from their menu-bar icons.
2. In **System Settings → VPN**, Tailscale should be **on** and every other VPN **off**.
3. In **System Settings → General → Login Items & Extensions → Network Extensions**, **Tailscale** should be on.
4. **Restart your Mac.** Open only Tailscale and wait for **Connected**.

## Test the connection

Paste into Terminal on **your own Mac** (not inside an SSH session to HG7):

```bash
TS=/Applications/Tailscale.app/Contents/MacOS/Tailscale; $TS ping -c 3 HG7_TAILSCALE_IP; route -n get HG7_TAILSCALE_IP | grep interface; nc -vz -G 5 HG7_TAILSCALE_IP 5900
```

What you should see:

- `pong from hg7-macmini`
- `interface: utun…`
- `succeeded`

| Result | Meaning | Fix |
| --- | --- | --- |
| All three good | The network works | Connect: `open vnc://HG7_TAILSCALE_IP` |
| `ping` fails or "no reply" | Wrong account/device, or Tailscale not really connected | Fix 2, then retry |
| `pong` but `interface:` is not `utun…` | Another app is taking HG7's traffic | Fix 3 |
| `pong` and `utun…` but `nc` times out | Another VPN or filter is blocking it | Fix 3. Also quit Docker/OrbStack and security tools, then retry |
| `nc` succeeded, but VNC says refused / not permitted | Not allowed to use Screen Sharing | Fix 1 (host administrator) |
| VNC logs in but spins forever | Session problem on HG7 | [TROUBLESHOOTING.md](TROUBLESHOOTING.md) section 4 |

A timeout before any password prompt is a **network** problem. Do not reset the password for it.

To open Screen Sharing from Terminal:

```bash
open vnc://HG7_TAILSCALE_IP
```

## Host administrator: check from HG7

On HG7, to see whether the coworker's Mac is online and reachable:

```bash
/Applications/Tailscale.app/Contents/MacOS/Tailscale status | grep COWORKER_TAILSCALE_IP
/Applications/Tailscale.app/Contents/MacOS/Tailscale ping COWORKER_TAILSCALE_IP
```

If it shows `offline` or "no reply" while the coworker's app says Connected, the coworker needs Fix 2.

## Still not working? Send this to the host administrator

A screenshot of the **Test the connection** output, plus:

```text
Date/time:
Tailscale account email shown in the menu:
Own Mac's Tailscale address:
hg7-macmini in the device list (yes/no):
Other VPNs/proxies running (names):
Test output: ping / interface / nc:
Exact VNC error or where it stops:
```

No passwords.

## What we know so far

- 2026-11-06: HG7's firewall is **off**, and the tailnet access policy **allows** the coworker's address on all ports.
- 2026-11-06: `tailscale ping` between the two Macs worked, but HG7 received **0 bytes** from the coworker's connection attempts. The traffic was going somewhere else on the coworker's Mac.
- 2026-11-06: the coworker's account was **not** in HG7's Screen Sharing allowed-users list (Fix 1).
- 2026-11-07: the coworker's Tailscale showed "Connected" with the other VPN off, but HG7 saw the coworker's only known device as **offline** (Fix 2).
- Earlier: the coworker logged in over SSH successfully once, after disconnecting NordVPN. A port-5900 test run *inside* HG7's SSH session only proves HG7 can reach itself.
- No successful coworker desktop session has been confirmed yet.

## More detail

- [CHECKLIST.md](CHECKLIST.md): the same fixes as a longer step-by-step list
- [INSTRUCTIONS.md](INSTRUCTIONS.md): first-time setup (accounts, sharing, SSH)
- [TROUBLESHOOTING.md](TROUBLESHOOTING.md): in-depth diagnosis, the SSH tunnel fallback, and the report template
- [AI_HANDOFF.md](AI_HANDOFF.md): upload this file to ChatGPT, or open the repository in Codex, for guided help

## References

- [Tailscale machine sharing](https://tailscale.com/docs/features/sharing)
- [Tailscale alongside other VPNs](https://tailscale.com/docs/reference/faq/other-vpns)
- [Apple Screen Sharing settings](https://support.apple.com/guide/mac-help/turn-screen-sharing-on-or-off-mh11848/mac)
- [Apple Remote Login settings](https://support.apple.com/guide/mac-help/allow-a-remote-computer-to-access-your-mac-mchlp1066/mac)

Menu names vary with macOS version and language.
