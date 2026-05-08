# OffCode Web Remote Control System 0.1 🌐


The **Web Remote Control System** is a high-performance, completely free, and self-hosted platform that lets you manage Windows devices directly from any web browser. Combining a lightweight Windows Agent with a modern Blazor-based control panel, you can monitor and control remote computers in real-time securely.

## ✨ Features
* **AES-256 GCM Encryption:** All sensitive data is encrypted end-to-end.
* **Live Stream & Interactive Control:** View the remote desktop and interact with the mouse/keyboard in real time.
* **Streaming File Manager:** Transfer large files efficiently without hitting memory limits.
* **Task Manager:** View active processes and manage them directly from the web interface.
* **Remote Process Execution:** Instantly run commands, open files, or launch software on the remote machine.
* **System Telemetry:** Monitor detailed hardware and software metrics (CPU, RAM, OS version) through the Computer Info dashboard.
* **Server Manager:** A built-in tool to easily configure ports, launch the server, and generate secure client agents (stubs).

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
   * **Server Address:** Your server's IP address or domain name.
   * **Frontend Port:** The port for the web dashboard (e.g., 5000).
   * **Backend Port:** The port for the API and SignalR communication (e.g., 5001).

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
**VER: 0.1**

* Fixed: Empty secret key generation issue in Server Manager.
* Fixed: Auto-detect project path and Server Manager crashing issues.

---
