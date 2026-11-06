# HG7 Mac mini remote access

Setup and troubleshooting for coworkers connecting through Tailscale, SSH, and macOS Screen Sharing (VNC).

**For ChatGPT or Codex: start with [AI_HANDOFF.md](AI_HANDOFF.md).** Download that file and upload it to ChatGPT, or open this repository in Codex.

**Screen Sharing times out? Start with [⭐ Try these first](CHECKLIST.md#-try-these-first-most-likely-fixes):**

1. Coworker: sign in to Tailscale with the exact email the share invitation went to (Log out → Quit → Log in), and check **hg7-macmini** is listed.
2. Host administrator: add the coworker to Screen Sharing's allowed users on HG7.
3. Coworker: quit other VPN/proxy apps completely, restart the Mac, open only Tailscale, try again.

Copy-paste Terminal commands for these checks: [CHECKLIST.md → Copy-paste commands](CHECKLIST.md#copy-paste-commands-coworkers-own-mac). Then work through the rest of [CHECKLIST.md](CHECKLIST.md) in order.

For manual setup, start with [INSTRUCTIONS.md](INSTRUCTIONS.md). For a failed connection, follow [TROUBLESHOOTING.md](TROUBLESHOOTING.md) in order.

## Connection details

Case notes updated 2026-11-06. Replace every uppercase placeholder before running a command; `coworkeraccount` and `youraccount` are examples, not real logins. Verify the real address in Tailscale.

| Item | Value |
| --- | --- |
| Machine | HG7 Mac Mini / `hg7-macmini` |
| Tailscale IP | `HG7_TAILSCALE_IP` |
| Full hostname | `HG7_FULL_TAILSCALE_HOSTNAME` |
| Project owner's Mac account | `youraccount` |
| Coworker's Mac account | `coworkeraccount` |
| SSH port | TCP 22 |
| Screen Sharing port | TCP 5900 |

The previous address, `OLD_TAILSCALE_IP`, is obsolete for this setup. Re-enrolling or replacing a Tailscale device can change its address and require a new share invitation.

No passwords, private keys, or invitation links belong in this repository. This public documentation repository does not configure machines or grant access. All host addresses and account names below are placeholders; obtain real values privately from the host administrator.

## What we actually know

- Project owner previously connected successfully using both SSH and Screen Sharing.
- Coworker successfully authenticated over SSH as `coworkeraccount`.
- The user reported SSH working after disconnecting NordVPN. Treat VPN coexistence as a likely contributor, not proof of every later failure.
- Coworker's successful port-5900 test was run **inside HG7's SSH session**. It proves HG7 could reach its own service, not that Coworker's computer could reach it.
- Coworker was reported to be in the Screen Sharing allowed-users list, but a check on HG7 on 2026-11-06 found the account was **not** a member. See [CHECKLIST.md](CHECKLIST.md) step 1.
- On 2026-11-06, `tailscale ping` between the two Macs succeeded while HG7 received 0 bytes from the coworker's connection attempts. HG7's firewall was off and the tailnet policy allowed the coworker's address.
- Coworker's Screen Sharing login still hung. The client-side port test and an end-to-end desktop login remain unverified. This repository does not claim that issue is resolved.

## References

- [Tailscale machine sharing](https://tailscale.com/docs/features/sharing)
- [Tailscale alongside other VPNs](https://tailscale.com/docs/reference/faq/other-vpns)
- [Apple Screen Sharing settings](https://support.apple.com/guide/mac-help/turn-screen-sharing-on-or-off-mh11848/mac)
- [Apple Remote Login settings](https://support.apple.com/guide/mac-help/allow-a-remote-computer-to-access-your-mac-mchlp1066/mac)

Menu names vary with macOS version and language. These instructions use ordinary macOS SSH over Tailscale networking, not the separate Tailscale SSH server feature.
