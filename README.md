# OffCode Web Remote Control System 0.7 🌐


The **Web Remote Control System** is a high-performance, completely free, and self-hosted platform that lets you manage Windows devices directly from any web browser. Combining a lightweight Windows Agent with a modern Blazor-based control panel, you can monitor and control remote computers in real-time securely.

## ✨ Features
* **AES-256 GCM Encryption:** All sensitive data is encrypted end-to-end.
* **Live Stream & Interactive Control:** View the remote desktop and interact with the mouse/keyboard in real time.
* **Chat:** Real-time two-way chat window between the dashboard administrator and the remote agent's user, complete with customizable administrator nickname and message history.
* **Share Screen (Public Viewer):** Generate a public, anonymous link to securely share the live stream of the remote device with others without giving them dashboard access.
* **Streaming File Manager:** Transfer large files efficiently without hitting memory limits; browse all ready drives (fixed, removable, network) and jump to common folders (e.g. Desktop, Documents, Downloads, Program Files, AppData); click-to-open navigation with upload target following the current path; optional **Upload and run** toggle to open uploaded files automatically on the agent; **File Search** by name with wildcard and recursive subdirectory support.
* **Desktop Screenshot:** Capture preview in the control panel; **Close Image** dismisses the preview without leaving the page.
* **Task Manager & Process Blocker:** View active processes and manage them directly from the web interface. Block specific applications by name so the remote agent instantly kills them upon startup. Setting persists even if the browser dashboard is closed.
* **Power Controls:** Shut down or restart the remote device from the Task Manager panel.
* **Software Manager:** List installed applications and uninstall them remotely using Windows Registry data.
* **Remote Process Execution:** Instantly run commands, open files, or launch software on the remote machine.
* **Remote Terminal & Script Manager:** Interactive PowerShell/CMD terminal directly in the browser, plus a built-in script editor. Write, save, and execute Python, PowerShell, Batch, and VBScript files securely on the remote agent. Advanced features include:
  * **Stop Script:** Safely and asynchronously terminate currently running scripts.
  * **Download Output:** Save script execution outputs directly to a local file.
  * **Scheduled Script Execution:** Set an exact date and time for scripts to run automatically on the agent, managed by the backend server even if the browser dashboard is closed.
* **System Telemetry:** Monitor detailed hardware and software metrics (CPU, RAM, OS version, network info) through the Computer Info dashboard. **Startup Programs** panel now includes a **Remove** button on each row — deletes the entry directly from the Windows Registry (`HKCU` / `HKLM`) or the startup folder. The agent must run as Administrator to remove `HKLM` entries; the result is reported back with a toast notification.
* **Device History:** Track and manage previously connected devices for quicker reconnection.
* **Agent Settings (Auto Startup Name):** In Agent settings, enabling Windows auto startup now opens a modal where you can set a custom startup name (default: `RemoteControlAgent`).
* **Dashboard Lock (Optional):** Password gate for the admin panel (`/authentication/login`) with settings at `/authentication/settings`. The password is stored only as a secure hash on the server. The Blazor app uses a **cookie** session; successful verification also issues a short-lived **JWT** (Bearer) for **SignalR** and protected **REST** APIs (`DashboardAccess`). Agent file-transfer HTTP endpoints (`agent-download` / `agent-upload`) stay **anonymous** so agents can pull/push files. Lock **configure** requires either an authenticated dashboard session or **`DASHBOARD_LOCK_ADMIN_SECRET`**. With lock **on**, unauthenticated users are not shown dashboard pages. With lock **off**, `/authentication/continue` can establish the session without a password. Lock settings include a **Dashboard** shortcut to the devices page (`/devices`).
* **Remove Agent:** Disconnect a device from the dashboard and trigger remote uninstall of the agent **executable** on the target PC; if the agent is offline, the request is **queued in SQLite** and runs when it reconnects.
* **Server Manager:** Configure ports, start the backend and frontend **independently** (stopping Frontend does not stop Backend, and vice versa), generate secure client agents (stubs), toggle optional file logging, and view the latest project changelog directly within the application.

---

## 🔐 Security Architecture

### 🛡️ Request Protection

| Layer | Protection |
|-------|------------|
| 1 | **Request timeout** — 30s global, 10 min for file transfers, ∞ for SignalR (Slowloris protection) |
| 2 | **Security headers** — CSP, HSTS, X-Frame-Options, X-Content-Type-Options, Referrer-Policy, Permissions-Policy, CORP, COOP |
| 3 | **Request audit trail** — All authentication events logged with IP, UserAgent, risk level and PII masking |
| 4 | **Kestrel hardening** — `RequestHeadersTimeout: 15s`, `KeepAliveTimeout: 120s`, HTTP/2 stream limit: 100 |
| 5 | **Login rate limiting** — 8 attempts per minute per IP+UserAgent (dashboard lock) |
| 6 | **API rate limiting** — 10 requests per minute per client (SecurityService) |

### 🔒 Server-Level Security

| Component | Implementation |
|-----------|----------------|
| **Transport encryption** | AES-256-GCM (SignalR payload E2E), TLS via reverse proxy (Cloudflare/Nginx) |
| **Password storage (Dashboard)** | ASP.NET Identity `IPasswordHasher` → PBKDF2-SHA256 with random salt |
| **JWT tokens** | HS256 (HMAC-SHA256), configurable expiry (5–240 min), mandatory strong key in production |
| **JWT signing key** | Enforced minimum 32 chars; weak/default keys blocked at startup in production mode |
| **API authentication** | JWT Bearer token with `DashboardAccess` claim required for all admin endpoints |
| **SignalR hub auth** | `DashboardHubAuthorizationFilter` — all methods require `dashboard_access: true` claim |
| **AES key validation** | App refuses to start if `AES_MASTER_KEY_HEX` is missing or not exactly 256 bits |
| **SQL injection** | Entity Framework Core parameterized queries only |
| **Error handling** | Internal exception details never exposed to HTTP clients; full details in server logs only |
| **Audit logging** | SQLite event log — action, IP, UserAgent, risk level (Low/Medium/High/Critical), PII-masked |
| **Brute force detection** | Automatic suspicious pattern detection (>10 failed logins or >5 rate-limit hits per hour) |
| **Timing-safe comparison** | `FixedTimeEquals` used for admin secret verification (constant-time XOR loop) |
| **CORS** | Configurable `ALLOWED_ORIGINS`; wildcard `*` blocked in production by default |

### 🖥️ FrontEnd-Level Security

| Component | Implementation |
|-----------|----------------|
| **Session cookie** | `RemoteControl.DashboardAuth`, SameSite=Strict, 8h expiry |
| **Blazor Server** | WebSocket-based — not vulnerable to traditional CSRF |
| **Return URL validation** | All `returnUrl` values validated — must start with `/`, no `//` open redirects |
| **XSS prevention** | Blazor renders via DOM — no raw `innerHTML` injection |
| **Content Security Policy** | Strict CSP on all pages; no unsafe-eval; limited external origins |

---

## 🛠️ Prerequisites

To run the Server Manager and compile the remote agent, you **must** have the .NET SDK installed on your system.
👉 [Download .NET 9.0 SDK](https://dotnet.microsoft.com/en-us/download/dotnet/9.0)

---

## 🚀 Getting Started

Follow these steps to set up your self-hosted remote control server and create your first client agent:

1. **Extract the files:** 
   Extract the downloaded `.zip` file completely into a folder. 
   
2. **Configure Server Details:**
   Open the **Server Manager** application. In the configuration settings, set the following:
   * **Server Address:** Your server's IP address or domain name (committed templates often use **`127.0.0.1`** for local development only).
   * **Frontend Port:** The port for the web dashboard (e.g., 5000).
   * **Backend Port:** The port for the API and SignalR communication (e.g., 5001).
   * **Encryption key:** Ensure **`AES_MASTER_KEY_HEX`** is set (64 hex chars) on BackEnd, FrontEnd, and agents before real use — repository samples may ship with an **empty** placeholder so secrets are not baked into the repo.

3. **Generate the Agent (Stub):**
   Click the **"Create remote agent file"** button in the Server Manager. This will compile a custom `.exe` file (stub) embedded with your specific server details and encryption keys. 
   * *Note: Running this generated agent file on a target Windows computer will establish the secure remote connection back to your web dashboard.*

---

## ⚠️ IMPORTANT WARNING

To ensure the system functions correctly, **DO NOT rename any folders or delete any files** within the extracted directory. The Server Manager relies on specific file paths and structures to compile the agent and run the backend/frontend servers. Any modifications to the directory structure will break the application.

---

## 🌐 Connectivity & Port Forwarding

If you intend to control devices over the internet (outside your local home network), you **must** expose your server so the agents can connect to it. You have several options:

1. **Cloudflare Tunnels (Highly Recommended):** The most secure and easiest method. No router port forwarding is required. Cloudflare Zero Trust connects your local ports to a domain name and automatically provides HTTPS/WSS.
2. **Port Forwarding (Router):** Manually configure your router to forward the Backend Port (and Frontend Port if you want dashboard access) to your server computer. You will need a public IP or Dynamic DNS (DDNS).
3. **Mesh VPNs (Tailscale / ZeroTier):** Create a secure virtual local network. Install the VPN on both the server and the target devices. No public internet exposure is needed.
4. **VPS Deployment:** Run the entire BackEnd and FrontEnd on a rented Ubuntu/Linux VPS with a reverse proxy like Nginx.

---

## 🔒 Security & Deployment

Because this is a self-hosted solution, you have full control over your data. For production deployment over the internet, it is highly recommended to use a reverse proxy (like Nginx or Cloudflare) to utilize HTTPS/WSS for an extra layer of transport security, in addition to the built-in AES-256 payload encryption.

### Nginx Configuration Example
If you are deploying behind Nginx on Windows or Linux, use the following template to properly route WebSocket (SignalR) and HTTP traffic. 
*Ensure your ServerManager is set to use ports 8001 and 8081 so Nginx can listen on 8000 and 8080.*

```nginx
events {
    worker_connections 1024;
}

http {
    # FRONTEND (Nginx listens on 8000 -> Routes to C# FrontEnd on 8001)
    server {
        listen 8000;
        server_name _; 

        location / {
            proxy_pass http://127.0.0.1:8001;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "Upgrade";
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }

    # BACKEND (Nginx listens on 8080 -> Routes to C# BackEnd on 8081)
    server {
        listen 8080;
        server_name _; 

        location / {
            proxy_pass http://127.0.0.1:8081;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "Upgrade";
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```
*Note: The C# backend and frontend natively support `X-Forwarded-For` and `X-Forwarded-Proto` headers, so they will automatically log the true client IP even behind the proxy.*


## ⚖️ Legal Disclaimer & Terms of Use

Please read and understand these terms before proceeding:

*   **Important Notice:** This software is intended only for lawful, authorized use. Do not use it for illegal activity, unauthorized access, harassment, or any other criminal behavior.
*   **User Responsibilities:** You are solely responsible for how you use this application and must comply with all applicable laws, policies, and permissions in your jurisdiction.
*   **Prohibited Uses:**
    *   Unauthorized access to systems or devices
    *   Harassment, stalking, or intimidation
    *   Any form of illegal surveillance
    *   Disrupting business operations without permission
    *   Violating privacy rights of others
    *   Any activity that violates local, state, or federal laws
*   **Liability Disclaimer:** Use at your own risk. The author(s) and distributor(s) assume no liability for misuse or any resulting damages, legal consequences, or other issues arising from the use of this software.

By downloading and using this software, you acknowledge that you have read, understood, and agree to be bound by these terms and conditions.

---

## 📄 License
This project is provided as a free tool for personal and commercial use.

https://www.dev-offcode.com/RemoteControl.html

---

## CHANGELOG:


**20/06/2026**
**VER: 0.7**

* Added: **Reverse Proxy Support** — Built-in support for Nginx and Cloudflare Tunnels utilizing `X-Forwarded-For` and `X-Forwarded-Proto` headers. The servers now seamlessly track real client IP addresses securely when placed behind a reverse proxy.
* Added: **Auto Startup Name modal** — Enabling Windows auto startup from Agent settings now opens a modal where you can choose a custom startup name (or keep default `RemoteControlAgent`).
* Improved: **Live Stream Cursor Visibility** — The real system cursor from the agent machine is now rendered directly into captured livestream frames, including scaled captures.
* Improved: **Live Stream Smoothness** — Streaming lag/stutter has been significantly reduced with pipeline optimizations, resulting in much smoother playback and fewer frame hiccups.
* Fixed: **Remote Control Right Click** — Right-click from dashboard now correctly executes as right-click on the agent side, and right mouse button no longer triggers left-button down/up drag events.


**18/06/2026**
**VER: 0.6**

* Security: **Request Timeout / Slowloris Protection** — `RequestTimeout` middleware added (30s global, 10 min for file transfers, ∞ for SignalR). Kestrel hardened with `RequestHeadersTimeout: 15s` and `KeepAliveTimeout: 120s`. HTTP/2 stream limit set to 100 per connection.
* Security: **Error Message Hardening** — Internal `ex.Message` no longer exposed in HTTP responses across `UserController`, `DeviceController`, and `FileManagerController` (12 locations fixed). All 500 responses now return a safe generic message; full exception details logged server-side only. Global fallback exception handler added to `BackEnd/Program.cs` with `correlationId` for log tracing.
* Fixed: **Script Execution Timeout Removed** — Scripts now run indefinitely until manually stopped by the user. The previous 5-minute hard timeout (`CancellationTokenSource(TimeSpan.FromMinutes(5))`) has been removed from `ScriptExecutionService`. Output buffer size limit also removed.
* Improved: **UI Modernization (Glassmorphism)** — The Blazor FrontEnd has been significantly upgraded with a sleek "glassmorphism" theme featuring emerald/teal accents. This applies to `LiveStream`, `DeviceControl`, `AgentSettingsPanel`, and authentication pages.
* Fixed: **Mobile Layout** — Connected devices and history views are now fully responsive on mobile devices via a robust Flexbox layout, fixing overflowing text and stretched elements.
* Improved: **Live Stream Viewer** — Added dynamic zoom controls allowing users to zoom in/out with the mouse wheel while tracking the cursor position. Added support for up to 240 FPS and dynamic quality settings (10-100%) for incredibly smooth playback, with 30 FPS set as the new default.
* Fixed: **Share Screen Link Copy** — The "Copy Link" button in the Live Stream view now uses synchronous JavaScript logic for improved reliability across all environments.
* Added: **Stealth Mode** — You can now hide the agent's tray icon directly from the "Agent settings" panel. When Stealth Mode is enabled, the agent runs silently in the background and is only visible in the Windows Task Manager.
* Added: **Chat** — Real-time two-way chat window between the dashboard administrator and the remote agent's user, complete with customizable administrator nickname and message history.
* Added: **Script Manager Enhancements (Job-based)** — The Remote Agent now tracks all running and completed scripts directly in its RAM. The dashboard automatically fetches this real-time status, ensuring you never lose track of a script's progress, even if you close the browser. Includes a new "Clear Recent" functionality, safe asynchronous script stopping, and the ability to download execution output.


**13/06/2026**
**VER: 0.5**


* Added: **Process Blocker** — Prevent specific applications from running on the remote device. Enter the process name in the Task Manager section, and the background agent service will instantly kill it whenever it attempts to start. The blocking setting is saved directly to the Windows Registry and persists across reboots and network reconnects.
* Added: **Startup Programs — Remove** — Each startup program entry in Computer Info now has a **Remove** button. Clicking it sends an encrypted `RequestRemoveStartupProgram` command through the hub to the agent, which deletes the registry key (`HKCU\..\Run`, `HKLM\..\Run`, or `Wow6432Node`) or the startup folder shortcut. The result is returned over the same encrypted channel and displayed as a toast notification (success / error). HKLM removal requires the agent to be running as Administrator.
* Added: **File Manager — Search** — Search for files by name in the current directory (including subdirectories) directly from the web dashboard. Results are capped at 500 items for safety and use the encrypted SignalR tunnel for communication.



**17/05/2026**
**VER: 0.4**

* Added: **Script manager tab / Scheduled Script Execution** - which allows setting a specific date and time for a script to run automatically, independently managed by the backend server.
* Added: **Server Manager Changelog** - In-app changelog viewer that fetches and parses the latest release notes automatically, along with a quick link to download new versions.

**14/05/2026**
**VER: 0.3**

* Added: **Share Screen** - Share the agent's live stream with anyone via a secure, public link without requiring a dashboard login. The public viewer connects anonymously while your dashboard acts as the decryption relay.
* Added: **Remote Terminal** - Fully interactive PowerShell and Command Prompt terminal directly in the web dashboard.
* Added: **Script Manager** - Built-in code editor to write, save, and execute scripts (Python, PowerShell, Batch, VBScript) on remote agent machines. Includes 

**09/05/2026**
**VER: 0.2**

* Added: **Software Manager** - Remotely list and uninstall applications via Windows Registry.
* Added: **Power Controls** - Shutdown and Restart devices remotely from the Task Manager panel.
* Added: **Device History** - Track and manage previously connected devices.
* Added: **File Manager — dynamic drives** — Quick-access buttons for all ready drives (fixed, removable, network) fetched from the agent over the encrypted SignalR channel.
* Added: **File Manager — quick folders** — Program Files, Program Files (x86), and Roaming **AppData** shortcuts (resolved on the agent via shell folder tokens, same pattern as Desktop/Documents/Downloads).
* Improved: **File Manager UX** — Open folders with a single click on the row; upload target automatically follows the current browse path (no separate “Set as Target” step).
* Added: **File Manager — upload and run** — Optional checkbox to open uploaded files automatically on the agent after transfer completes.
* Added: **Desktop Screenshot — Close Image** — Dismiss the screenshot preview from the device control UI.
* Added: **Server Manager — log toggle** — Checkbox to turn application logging on or off.
* Fixed: **Server Manager** — Stopping or restarting the **frontend** no longer stops the **backend** (independent service controls).
* Added: **Dashboard Lock (optional)** — Cookie session for the Blazor UI, lock-aware redirects, `/authentication/continue` when lock is disabled, safe `returnUrl` handling; backend stores the dashboard password as a secure hash; short-lived **JWT** for admin **SignalR** and protected **REST** (`DeviceController`, admin `FileManager` routes); agent HTTP file endpoints (`agent-download` / `agent-upload`) remain reachable without JWT; **`configure`** requires auth or **`DASHBOARD_LOCK_ADMIN_SECRET`**.
* Config samples: **`AES_MASTER_KEY_HEX`** default empty and example URLs use **`127.0.0.1`** in committed `.env` / `appsettings` templates (set real values for production).
* Added: **Remove Agent** — Remote uninstall flow with confirmation; **offline queue** in SQLite (`PendingAgentCommands`) until the agent reconnects.
* Added: **Lock settings UI** — **Dashboard** button on `/authentication/settings` opens the devices page (`/devices`).


**08/05/2026**
**VER: 0.1**

* Fixed: Empty secret key generation issue in Server Manager.
* Fixed: Auto-detect project path and Server Manager crashing issues.

---
