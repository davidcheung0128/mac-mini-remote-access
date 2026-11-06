# What to try, in order

A short checklist for the current Screen Sharing timeout. Stop at the first step that fixes it. Details and background are in [TROUBLESHOOTING.md](TROUBLESHOOTING.md). Replace the uppercase placeholders with the real values you were given privately.

## ⭐ Try these first (most likely fixes)

1. **Coworker: sign in to Tailscale with the invited email.** Tailscale menu-bar icon → check that the email at the top is the one the share invitation went to. If it is different: **Log out → Quit → reopen → Log in with that email**, and make sure **hg7-macmini** is in the list. Details: [step 2a](#2a-connected-but-nothing-works-check-the-account-and-device).
2. **Host administrator: add the coworker to Screen Sharing.** On HG7: System Settings → General → Sharing → ⓘ Screen Sharing → **+** → the coworker's account. Without this, the login is refused even when the network works. Details: [step 1](#1-allow-the-coworker-to-use-screen-sharing).
3. **Coworker: quit other VPN/proxy apps completely and restart your Mac.** After restarting, open only Tailscale and try `vnc://HG7_TAILSCALE_IP` again. Details: [steps 4–5](#4-quit-other-vpn-and-proxy-apps-completely).

Still not working? Go through the full list below in order.

## Findings from HG7's side (2026-11-06)

- HG7's macOS firewall is **off**, so it is not blocking anything.
- The tailnet access policy **allows** the coworker's Tailscale address on all ports.
- `tailscale ping` between HG7 and the coworker's Mac **succeeds**. Tailscale itself is connected.
- During the coworker's SSH/VNC attempts, HG7 **received 0 bytes** from the coworker's Mac. The connection attempts never reached HG7, so the coworker's Mac is sending them somewhere other than Tailscale.
- The coworker's account was **not yet** in Screen Sharing's allowed-users list. Even after the network works, VNC will refuse the login until the host administrator fixes this (step 1).
- On 2026-11-07, the coworker's Tailscale app showed "Connected" with the other VPN off, but HG7 saw the coworker's only known device as **offline** and could not ping it. The Mac was connected to Tailscale, but not as the device or account that HG7 is shared with. Fix: step 2a.

## Host administrator

### 1. Allow the coworker to use Screen Sharing

On HG7, using an admin account, do one of these:

- **System Settings → General → Sharing → ⓘ next to Screen Sharing → Allow access for: Only these users → + → the coworker's account → Done.**
- Or in Terminal on HG7:

  ```bash
  sudo dseditgroup -o edit -a coworkeraccount -t user com.apple.access_screensharing
  ```

Check it worked (prints "user is a member of the group"):

```bash
dsmemberutil checkmembership -U coworkeraccount -G com.apple.access_screensharing
```

Remote Login (SSH) has its own separate list. Being allowed for SSH does not allow Screen Sharing.

## Coworker, on your own Mac

Run every command in a **new Terminal window on your own Mac**, not inside an SSH session to HG7.

### 2. Make sure Tailscale is connected and HG7 is visible

Click the Tailscale menu-bar icon. It should say **Connected**, signed in with the same email the share invitation went to, and **hg7-macmini** should be in the list. Use the address shown there for HG7.

```bash
/Applications/Tailscale.app/Contents/MacOS/Tailscale ping HG7_TAILSCALE_IP
```

`pong from …` means Tailscale works. If it fails, or "Connected" shows but nothing works, do step 2a.

### 2a. "Connected" but nothing works: check the account and device

"Connected" only means your Mac is on *a* tailnet. HG7 is shared with one specific Tailscale account. If your Mac is signed in with a different email, or is a different or re-registered device, HG7 cannot see it.

**Check:**

1. Click the Tailscale menu-bar icon.
2. The **account email** at the top must be exactly the email the share invitation was sent to (`COWORKER_INVITED_EMAIL`).
3. Under **This device**, note your Mac's Tailscale address (`COWORKER_TAILSCALE_IP`).
4. **hg7-macmini** must appear in the device list.
5. In Terminal:

   ```bash
   /Applications/Tailscale.app/Contents/MacOS/Tailscale status
   ```

   The first line is your own Mac: its address and account. Check that HG7 appears in the list and isn't marked `offline`.

**Fix, if the email is wrong, HG7 is missing, or the host administrator says your device looks offline:**

1. Tailscale menu-bar icon → your account → **Log out**.
2. Quit Tailscale completely (menu-bar icon → **Quit**).
3. Open Tailscale again → **Log in** → choose the **same sign-in method and email** the invitation was sent to (e.g. "Sign in with Google" for a Gmail address).
4. If you have several accounts, use the account switcher in the menu to select that email.
5. If **hg7-macmini** is still missing, open the invitation email again and click **Accept** while signed in with that email. Ask the host administrator to resend the share if the link has expired.
6. Wait until the icon shows **Connected** and **hg7-macmini** is in the list.
7. Send the host administrator your new `COWORKER_TAILSCALE_IP`, which may have changed. They can confirm from HG7 that your device is online:

   ```bash
   /Applications/Tailscale.app/Contents/MacOS/Tailscale status | grep COWORKER_TAILSCALE_IP
   /Applications/Tailscale.app/Contents/MacOS/Tailscale ping COWORKER_TAILSCALE_IP
   ```

8. Run the `tailscale ping` in step 2, then step 3.

If the email and device are correct but the host administrator still sees your device as offline, restart your Mac (step 5), open only Tailscale, and check again.

### 3. Find out where your Mac sends HG7 traffic

```bash
route -n get HG7_TAILSCALE_IP | grep interface
nc -vz -G 5 HG7_TAILSCALE_IP 5900
```

| Result | Meaning | Go to |
| --- | --- | --- |
| `interface: utun…` and `nc` **succeeded** | The network works | Step 7 |
| `interface:` is **not** `utun…` (e.g. `en0`, `bridge100`) | Another app is taking over HG7's address | Steps 4–5 |
| `utun…` but `nc` **times out** | Another VPN's `utun` may own the route, or a filter blocks it | Steps 4–5 |

### 4. Quit other VPN and proxy apps completely

Quit NordVPN, Clash/Surge-type proxies, company VPNs and similar tools from their menu-bar icons. Switching them off is not always enough, because they can leave routes behind.

Then check:

- **System Settings → VPN:** Tailscale **on**, every other VPN **off**.
- **System Settings → General → Login Items & Extensions → Network Extensions:** **Tailscale** on.

Run step 3 again.

### 5. Restart your Mac

A restart clears routes left behind by other apps. After it restarts, open **only Tailscale**, wait for Connected, and run step 3 again.

### 6. If it still fails

- Docker Desktop, OrbStack and some security software also create network routes. Quit them and run step 3 again.
- If it works only with your other VPN off and you need that VPN for work, that is a VPN coexistence issue. See [Tailscale with other VPNs](https://tailscale.com/docs/reference/faq/other-vpns) and ask the network administrator. Do not disable firewalls or the kill switch as a blanket fix.
- Send the host administrator a screenshot of the step 3 output, together with the report template in [TROUBLESHOOTING.md](TROUBLESHOOTING.md). No passwords.

### 7. Connect with Screen Sharing

1. Finder → **Command+K** → `vnc://HG7_TAILSCALE_IP` → Connect.
2. Sign in with your **HG7 account** name and password (not your Tailscale or own-Mac password).
3. If asked, choose **Log in as yourself**.

| What you see | Cause | Fix |
| --- | --- | --- |
| Times out before any password prompt | Network (steps 2–6) | Not the password. Do not reset it |
| Password prompt, then refused or "not permitted" | Not in the Screen Sharing list | Step 1 (host administrator) |
| Logs in but spins forever | Session problem | [TROUBLESHOOTING.md](TROUBLESHOOTING.md) section 4 |

When you finish, close the window or log out. **Never Restart or Shut Down HG7**: it runs shared work systems that only come back after the host account logs in again.
