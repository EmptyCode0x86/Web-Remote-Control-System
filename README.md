# OffCode Web Remote Control System 0.2 🌐


The **Web Remote Control System** is a high-performance, completely free, and self-hosted platform that lets you manage Windows devices directly from any web browser. Combining a lightweight Windows Agent with a modern Blazor-based control panel, you can monitor and control remote computers in real-time securely.

## ✨ Features
* **AES-256 GCM Encryption:** All sensitive data is encrypted end-to-end.
* **Live Stream & Interactive Control:** View the remote desktop and interact with the mouse/keyboard in real time.
* **Streaming File Manager:** Transfer large files efficiently without hitting memory limits; browse all ready drives (fixed, removable, network) and jump to common folders (e.g. Desktop, Documents, Downloads, Program Files, AppData); click-to-open navigation with upload target following the current path; optional **Upload and run** toggle to open uploaded files automatically on the agent.
* **Desktop Screenshot:** Capture preview in the control panel; **Close Image** dismisses the preview without leaving the page.
* **Task Manager:** View active processes and manage them directly from the web interface.
* **Power Controls:** Shut down or restart the remote device from the Task Manager panel.
* **Software Manager:** List installed applications and uninstall them remotely using Windows Registry data.
* **Remote Process Execution:** Instantly run commands, open files, or launch software on the remote machine.
* **System Telemetry:** Monitor detailed hardware and software metrics (CPU, RAM, OS version) through the Computer Info dashboard.
* **Device History:** Track and manage previously connected devices for quicker reconnection.
* **Dashboard Lock (Optional):** Password gate for the admin panel (`/authentication/login`) with settings at `/authentication/settings`. The password is stored only as a secure hash on the server. The Blazor app uses a **cookie** session; successful verification also issues a short-lived **JWT** (Bearer) for **SignalR** and protected **REST** APIs (`DashboardAccess`). Agent file-transfer HTTP endpoints (`agent-download` / `agent-upload`) stay **anonymous** so agents can pull/push files. Lock **configure** requires either an authenticated dashboard session or **`DASHBOARD_LOCK_ADMIN_SECRET`**. With lock **on**, unauthenticated users are not shown dashboard pages. With lock **off**, `/authentication/continue` can establish the session without a password. Lock settings include a **Dashboard** shortcut to the devices page (`/devices`).
* **Remove Agent:** Disconnect a device from the dashboard and trigger remote uninstall of the agent **executable** on the target PC; if the agent is offline, the request is **queued in SQLite** and runs when it reconnects.
* **Server Manager:** Configure ports, start the backend and frontend **independently** (stopping Frontend does not stop Backend, and vice versa), generate secure client agents (stubs), and toggle optional file logging with a single checkbox.

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
