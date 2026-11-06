# Diagnose and resolve connection failures

Work through these checks in order. Record results instead of repeatedly retrying the same login. Use the current address in [README.md](README.md), after confirming it in Tailscale.

## 1. Confirm which computer is running the command

Open a **new Terminal window on the coworker's own Mac**. Run:

```bash
hostname
whoami
```

If the prompt says `coworkeraccount@HG7-Mac-Mini`, you are already inside SSH on HG7. Type `exit` before testing connectivity from Coworker's Mac. A successful HG7-to-itself test does not establish remote reachability.

## 2. Test network access from the coworker's Mac

Keep Tailscale connected. Confirm HG7 is shared to the same Tailscale account. For the controlled VPN test, disconnect NordVPN temporarily if permitted.

```bash
nc -vz -G 10 HG7_TAILSCALE_IP 22
nc -vz -G 10 HG7_TAILSCALE_IP 5900
```

These are macOS commands and do not submit a password or alter the server.

| Result | Meaning and next action |
| --- | --- |
| Both succeed | TCP connections are possible. Proceed to authentication/session checks; this does not prove a healthy desktop session. |
| Both time out | Check current address, invitation/account, HG7 power and internet, both Tailscale clients, VPN interference, and access policy. |
| SSH succeeds, 5900 times out | Investigate Screen Sharing-specific policy/firewall/service access. Ask Host administrator to verify the service and the administrator to check TCP 5900 access. |
| Connection refused | The destination or a firewall actively rejected the connection. Check the service and port; do not reset passwords to fix this. |

Inspect Tailscale status on the coworker's Mac:

```bash
tailscale status
```

If the command is unavailable, use the app, or with the macOS app installed:

```bash
/Applications/Tailscale.app/Contents/MacOS/Tailscale status
```

A green status icon alone does not prove SSH or VNC traffic is getting through. If a VPN-off test succeeds and VPN-on fails, preserve those results and work with the VPN/network administrator on supported coexistence settings. Do not blindly add routes or disable the kill switch. [Tailscale VPN guidance](https://tailscale.com/docs/reference/faq/other-vpns).

## 3. Test authentication independently

From the coworker's Mac:

```bash
ssh -o ConnectTimeout=10 coworkeraccount@HG7_TAILSCALE_IP
```

- **Successful shell login:** the credentials worked for SSH at that time. Screen Sharing has its own authorization settings.
- **Permission denied:** verify the account name, password, and Remote Login allowed-users list with Host administrator.
- **Timeout before password:** investigate networking/service reachability, not the password.
- **Host key warning:** verify the fingerprint with Host administrator; do not bypass checking or erase known-host entries indiscriminately.

For more detail, retry once with `ssh -v -o ConnectTimeout=10 coworkeraccount@HG7_TAILSCALE_IP`. Share only relevant error lines, removing private paths or unrelated host details.

## 4. SSH works but Screen Sharing spins

1. Cancel the stuck attempt and quit the Screen Sharing client normally. Reopen one connection, not several concurrent attempts.
2. Use **Standard** mode for the test. High Performance is a separate variable, not a confirmed cause of this incident.
3. Use `coworkeraccount` and the same current HG7 password that worked for SSH.
4. Host administrator confirms that exact account is in the Screen Sharing allowed-users list, even if Remote Login already allows it.
5. Host administrator checks that Coworker can sign into his account locally on HG7, without terminating another person's work. Complete any first-login setup if presented. A local failure points to an account/session problem rather than Tailscale.
6. If TCP 5900 works but the application still hangs, arrange with Host administrator to restart Screen Sharing by toggling it off and back on locally. This interrupts screen-sharing connections; Host administrator should be physically present or have another working management path. Do not change Remote Management configuration casually.
7. Only if the problem persists, arrange a Mac restart after all users save work. Do not reboot a shared Mac without coordinating.

After each change, retry once and record the result. Stop when your own desktop loads and responds.

### Optional server-side check

Inside a working SSH session on HG7:

```bash
nc -vz -G 3 127.0.0.1 5900
```

Success shows a listener on HG7's loopback port. Failure there suggests checking the local sharing service. This check complements, never replaces, the client-side test in step 2.

### Optional Standard VNC connection over an SSH tunnel

Use only if Host administrator authorizes Screen Sharing for your account and the network administrator permits forwarding. Do not use this to bypass an intentional VNC access restriction. It can help isolate a direct-port networking problem when SSH works.

Run on **Coworker's own Mac**:

```bash
ssh -N -o ExitOnForwardFailure=yes -o ConnectTimeout=10 -L 127.0.0.1:15900:127.0.0.1:5900 coworkeraccount@HG7_TAILSCALE_IP
```

Authenticate and leave this Terminal open; no shell prompt is expected with `-N`. In Finder → Command+K, connect to `vnc://127.0.0.1:15900` using Standard mode and your HG7 credentials. The listener is restricted to your own computer. Press Control+C in the tunnel Terminal when finished.

If this works while direct Standard VNC fails, investigate the direct 5900 path. If it still hangs, it does not fix server-side authentication/session problems. If SSH reports forwarding prohibited, ask the administrator; do not override their policy.

## 5. Current incident: next missing evidence

For the short ordered list of fixes, see [CHECKLIST.md](CHECKLIST.md). As of 2026-11-06, HG7 received no traffic from the coworker's Mac even though `tailscale ping` worked, which points to routing on the coworker's Mac (another VPN or proxy).

Coworker authenticated over SSH. HG7's self-test on port 5900 succeeded. The remaining required test is **Coworker's Mac → HG7 TCP 5900**, followed by a Standard-mode login with the verified account. No successful Coworker desktop session has been documented yet.

## Report template

```text
Date/time and time zone:
Client computer and macOS version:
HG7 current Tailscale IP:
HG7 account name (no password):
NordVPN/other VPN connected or disconnected:
Command ran on client or inside SSH:
Client TCP 22 test:
Client TCP 5900 test:
SSH authentication successful:
Screen Sharing Standard/High Performance:
Account in Screen Sharing allowed-users list:
Exact error or stage where it hangs:
Last change and result:
```

Do not include passwords, private keys, authentication URLs, or screenshots of another user's desktop. A local `.zshrc` warning about a missing OpenClaw completion file is separate from SSH authentication; do not confuse it with the remote connection error.
