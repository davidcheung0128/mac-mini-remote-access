# Set up and connect

## 1. Host administrator: prepare the HG7 Mac mini

Do these steps on **HG7**, using an administrator account.

1. Keep the machine awake and connected to the internet during setup and testing.
2. Open Tailscale and confirm it is connected. Record the current IP from its device details.
3. In **System Settings → Users & Groups**, create a separate **Standard** account for each coworker. Record its account name (for example, `coworkeraccount`), not just its display name. Give the initial password to that person privately.
4. In **System Settings → General → Sharing**, enable **Remote Login**. Open its information/settings panel and allow the coworker's account. This enables SSH.
5. In the same Sharing pane, enable **Screen Sharing**. Open its information panel and add the coworker under **Allow access for → Only these users**. SSH permission and Screen Sharing permission are separate.
6. If **Remote Management** is already enabled, check with the machine administrator before changing it. Do not disable an existing management service just to follow this guide.
7. In Tailscale's admin console, select HG7 and share it with the coworker using an email invitation or private share link. The coworker must accept using the account signed into their Tailscale app. A machine-detail URL is not an invitation.

Tailscale sharing provides network access, not a macOS login account. Access policies still apply. No public router port forwarding is required. See [Tailscale sharing](https://tailscale.com/docs/features/sharing) and [Apple's sharing settings](https://support.apple.com/guide/mac-help/turn-screen-sharing-on-or-off-mh11848/mac).

## 2. Coworker: prepare your own computer

1. Install and sign into Tailscale, accept Host administrator's invitation, and connect.
2. Confirm HG7 appears in your Tailscale device list.
3. For the initial test, temporarily disconnect NordVPN or another overlapping VPN if your workplace permits it; keep Tailscale connected. Test with one network configuration at a time. Do not disable firewalls or security controls as a blanket fix. [VPN coexistence guidance](https://tailscale.com/docs/reference/faq/other-vpns).
4. Use your **HG7 account password**, not your Tailscale password or your own computer's password.

## 3. Connect with SSH

Run in **Terminal on your own computer**. Coworker:

```bash
ssh -o ConnectTimeout=10 coworkeraccount@HG7_TAILSCALE_IP
```

Project owner:

```bash
ssh -o ConnectTimeout=10 youraccount@HG7_TAILSCALE_IP
```

On the first connection, verify the host fingerprint with Host administrator before accepting it. If necessary, Host administrator can inspect the ED25519 fingerprint on HG7 with:

```bash
ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub
```

Type the full word `yes` only after verification. Type your password when prompted; Terminal displays no characters as you type it.

After login, confirm your identity and machine:

```bash
whoami
hostname
```

Coworker should see `coworkeraccount` and HG7's hostname. Return to your own computer with:

```bash
exit
```

Do not ignore a changed-host-key warning. Ask Host administrator whether HG7 was rebuilt and verify the new fingerprint before updating a saved key.

## 4. Connect to your own desktop with Screen Sharing

On the coworker's **Mac**:

1. Open Finder and press **Command+K**.
2. Enter `vnc://HG7_TAILSCALE_IP` and connect.
3. Choose **As Registered User**, enter your HG7 account name and password.
4. If offered a choice, select **Log in as yourself** for your own account, then Connect. Do not select **Share the display** when you want your own session.
5. For troubleshooting, use **Standard** mode: in Screen Sharing's saved-connections list, open the connection's information panel and change **Screen Sharing Type** if that option is available.

Success means your desktop actually appears and responds; reaching a password dialog or an open network port is not sufficient.

### Privacy limitation

Choosing your own session is different from enforcing a security rule that prevents access to every other session. We have not verified a built-in setting that removes the Share the display option while preserving personal-session access. The allowed-users list controls who may connect; it is not an own-session-only switch. Do not promise that Standard account status alone removes that option. If strict separation is required, have the administrator design an isolated environment such as a dedicated machine or separately managed VM.

## 5. Using a coding assistant through SSH

A coding assistant running on your own computer can use an authorized SSH connection to execute remote commands. This does not require VNC or installing the assistant on HG7. Support for a native remote-connection UI depends on the application.

Example request:

> Connect through SSH to coworkeraccount@HG7_TAILSCALE_IP. First run whoami, hostname, and pwd. Identify my project folder before editing anything. Use only my account and do not change system or sharing settings.

For an SSH connection form:

| Field | Value |
| --- | --- |
| Name | HG7 Mac mini |
| Host | `coworkeraccount@HG7_TAILSCALE_IP` (Project owner uses `youraccount`) |
| Port | `22` |
| SSH key | Leave blank unless a working key is already configured |
| Project/worktree folder | Your actual remote project folder, or the application's default |

A key field takes a private-key **path**, never a password. This task has not configured SSH keys. If the app cannot prompt for passwords, arrange key-based access with Host administrator rather than putting a password in a script. Never copy private keys onto HG7 or into this repository.

## Short Cantonese setup message

可唔可以喺 HG7 Mac mini 幫同事開一個獨立嘅標準使用者帳戶？然後去「系統設定 → 一般 → 共享」，喺「遠端登入」同「螢幕共享」兩邊都加返佢做允許使用者。再用 Tailscale 分享部機俾佢。帳戶名、密碼同部機地址私下俾佢，唔好放喺公開 repo。測試嗰陣保持部機醒住，唔該！
