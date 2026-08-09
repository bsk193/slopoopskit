# PS5 Jailbreak Host

A clean, professional browser-based jailbreak for PlayStation 5 firmware **9.00 – 12.70**.

## Overview

This is a self-hosted exploit chain that runs entirely in the PS5 browser. It uses a WebKit vulnerability to escape the browser sandbox, then escalates to kernel privileges via one of three firmware-specific exploits.

| Firmware | Exploit | Time | File |
|----------|---------|------|------|
| 9.00 – 10.01 | **Lapse** (AIO double-free) | Seconds | `src/lapse-ps5.js` |
| 10.20 – 12.00 | **Poops** (NetControl IPv6 UAF) | ~30s – 2min | `src/netctrl-ps5.js` |
| 12.02 – 12.70 | **P2JB** (cr_refcnt overflow) | ~50 minutes | `src/netctrl-ps5.js` |

## Quick Start

### Option A: GitHub Pages (Easiest)

This repository includes a GitHub Actions workflow that automatically deploys to GitHub Pages on every push to `main`.

**Setup:**
1. Fork or push this repo to GitHub
2. Go to **Settings → Pages**
3. Set **Source** to "GitHub Actions"
4. Push any commit to `main` (or trigger the workflow manually)
5. The site will be available at `https://yourusername.github.io/repo-name/`

**PS5 DNS Setup:**
- Point `manuals.playstation.net` to your GitHub Pages URL
- Or use a local DNS server / Pi-hole to redirect

### Option B: Self-Hosted

Serve this directory over **HTTPS** (port 443). The PS5 browser accepts a self-signed certificate for `manuals.playstation.net`.

**Requirements:**
- DNS spoof `manuals.playstation.net` to your server IP
- HTTPS on port 443
- Files served from `/document/en/ps5/`

### Using on PS5

1. Configure DNS to point `manuals.playstation.net` to your host
2. Open **User's Guide** on the PS5
3. The landing page auto-detects your firmware and shows support status
4. Click **RUN** — the exploit chain executes automatically
5. Do not close the browser window during execution

## GitHub Actions

The included workflow (`.github/workflows/deploy.yml`) automatically:
- Verifies all required files are present
- Deploys to GitHub Pages on every push to `main`

**To enable:**
1. Go to repository **Settings → Pages**
2. Under **Build and deployment**, select **Source: GitHub Actions**
3. The workflow will run automatically on your next push

## File Structure

```
.
├── index.html              # Landing page (firmware detection, status, run)
├── netctrl.html            # Exploit runner for Poops / P2JB chain
├── notify.html             # WebKit-only fallback for unsupported firmwares
├── src/
│   ├── netctrl-ps5.js      # Poops / P2JB kernel exploit (10.20 – 12.70)
│   ├── lapse-ps5.js        # Lapse kernel exploit (9.00 – 10.01)
│   ├── lapse-runtime.js    # WebKit runtime (syscalls, ROP, memory primitives)
│   ├── rop-worker.js       # Sacrificial Worker thread for ROP chains
│   ├── rop_slave.js        # Worker bootstrap
│   └── aioshellcode.js     # ELF loader injection
├── offsets/
│   ├── offsets.json          # Slopkit WebKit offsets per firmware
│   ├── lapse-offsets.json    # Kernel exploit gadget tables
│   └── extract-gadgets.py    # Tool to regenerate offset tables from firmware
├── elfldr-ps5-1360.elf     # ELF loader payload
└── kexp_2026_05_25.bin     # Kernel exploit shellcode blob
```

## How It Works

1. **WebKit Sandbox Escape** (`lapse-runtime.js`)  
   Corrupts `history.state` to gain arbitrary read/write in the browser process.

2. **Runtime Initialization** (`lapse-runtime.js`)  
   Builds syscall, ROP, and memory primitives on top of the WebKit exploit.

3. **Kernel Exploit** (`netctrl-ps5.js` or `lapse-ps5.js`)  
   Uses the runtime to trigger the firmware-specific kernel bug and gain kernel R/W.

4. **Jailbreak** (`netctrl-ps5.js`)  
   Patches `ucred`, breaks the sandbox, and enables debug settings.

5. **ELF Loader** (`aioshellcode.js`)  
   Injects the ELF loader into kernel memory. Homebrew payloads can then be sent to port **9021**.

## Important Notes

- **Reboot between failed attempts.** A lost race can leave a corrupted `ucred` in memory. Re-running without rebooting may panic the console.
- **Poops is a race exploit.** It may take 1–3 attempts to win. The code limits attempts to prevent hangs.
- **P2JB is deterministic but slow.** It requires ~50 minutes of `setuid()` calls to overflow `cr_refcnt`.

## Credits

- **[TheFloW](https://github.com/TheOfficialFloW)** — IPv6 UAF kernel exploit (Poops)
- **[egycnq](https://github.com/egycnq)** — Poopsploit implementation
- **[Gezine](https://github.com/Gezine)** — BD-JB5, Luac0re, Y2JB
- **[zecoxao](https://github.com/zecoxao)** — slopkit2 runtime porting
- **[jordyidk](https://github.com/jordyidk)** — Slopkit WebKit launcher

## License

This project is provided for **educational and research purposes only**.  
The original works retain their respective licenses.

---

**Disclaimer:** Use at your own risk. Modifying console firmware may void your warranty and violate terms of service.
