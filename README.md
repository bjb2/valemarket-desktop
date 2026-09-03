# ValeMarket Desktop

> **Preferred install:** New users should install [Vale Companion](https://github.com/bjb2/valecompanion/releases/latest), which combines ValeMarket and ValeLoot in one passive desktop application. This standalone ValeMarket repository remains available for existing installations.

A Windows and Linux desktop market browser and passive community contributor for **Spirit Vale**.

ValeMarket shows currently active listings reported by contributors. It is not a sale-history tracker, does not automate the game, and does not modify game files.

![ValeMarket Desktop browsing live Spirit Vale listings](docs/market.webp)

- Browse the live market at [market.spiritvalers.com](https://market.spiritvalers.com/)
- Integrate with the [public market API](https://market.spiritvalers.com/api/)
- Download the latest desktop build from [Releases](https://github.com/bjb2/valemarket-desktop/releases/latest)
- Report bugs through [GitHub Issues](https://github.com/bjb2/valemarket-desktop/issues)

## Install

### Windows

Requirements:

- Windows 10 or 11, x64
- [Npcap](https://npcap.com/#download)
- Spirit Vale

Steps:

1. Download `ValeMarket-Desktop-v<version>-windows-x64.zip` from the latest release.
2. Extract the entire archive to a folder you can keep.
3. Run `ValeMarket Desktop.exe` from that folder.

Keep the extracted folder together. ValeMarket stores its settings, logs, and contributor identity in its `data` directory.

The initial build is unsigned, so Windows may show an unknown-publisher warning. Verify that the download came from this repository and compare its SHA-256 checksum with the release notes.

### Linux

Requirements:

- x64 Linux with glibc
- libpcap
- dumpcap from Wireshark (recommended) or `CAP_NET_RAW` and `CAP_NET_ADMIN` on the packaged Bun executable
- Spirit Vale running natively or through Proton

Download the AppImage or `.tar.gz` build from the latest release. Mark the AppImage executable before launching it:

```sh
chmod +x ValeMarket-Desktop-v<version>-linux-x64.AppImage
./ValeMarket-Desktop-v<version>-linux-x64.AppImage
```

Automatic capture prefers a working `dumpcap` installation and falls back to direct libpcap. The Contributor panel can force either backend for diagnosis. Set `VALEMARKET_DUMPCAP` to a non-standard dumpcap path or `VALEMARKET_PCAP_LIBRARY` to a non-standard libpcap shared library. Linux settings and logs use the Electron user-data directory.

## Market refresh

ValeMarket loads one complete public snapshot generated every 15 minutes. Item inspectors fetch a rolling seven-day series of hourly observed asking-price quartiles. These are listing observations, not completed sales.

- Select **Refresh** in the header to check the current edge snapshot.
- The desktop app refreshes automatically every 30 minutes while visible.
- Returning to a window that has been in the background for at least 30 minutes also refreshes it.
- Search text, filters, sorting, and the selected item remain in place across a refresh.

The displayed timestamp comes from the API snapshot. Expired listings are filtered again in the client and never shown, even if they expired during the cache window.

## Passive contribution

Contribution is enabled by default. Open **Contributor** in the header to turn it off or select a capture adapter. Automatic uses the active default-route adapter and restarts capture when that route moves to another adapter.

When Npcap (Windows) or libpcap/dumpcap (Linux) is available, ValeMarket observes Spirit Vale market traffic locally and uploads normalized listing observations to the community API. New contributors are eligible immediately; a single eligible contributor is sufficient for the launch policy.

ValeMarket reassembles fragmented LiteNetLib messages before FishNet decoding, including large market result pages. Starting ValeMarket before Spirit Vale preserves the full session setup. If Spirit Vale is detected but no attributed traffic reaches the resolved adapter, the Contributor panel identifies that adapter and prompts you to select the active tunnel or route-optimizer adapter directly. If the panel reports unresolved linked responses, copy the diagnostic state and report it; the spawn, registration, and session-link counters contain no packet payload or account data. If it reports incomplete fragmented messages, select the adapter carrying Spirit Vale traffic directly.

Uploads include the listing price, quantity, status, displayed stats, and visible refine, potential, card, gem, and gem-refine metadata.

ValeMarket does **not** upload:

- raw network packets
- account or character identifiers
- seller or buyer identities
- installation paths
- unrelated application traffic

If another tool already contributes to ValeMarket, enable contribution in only one application.

## Local diagnostics

ValeMarket writes structured JSON Lines logs to `data/logs` in the extracted Windows portable build. Other builds use the platform application-data directory. Records cover application, capture backend, packet decoding, persistence, and contributor upload lifecycle events. Raw packets, installation tokens, listing payloads, and seller or buyer data are redacted.

Logs rotate at 2 MiB. ValeMarket retains at most 20 files and 20 MiB for 14 days. Open **Contributor** and select **Export diagnostics** to create a support folder containing the current session logs, a privacy-safe desktop state snapshot, and a manifest. The five newest diagnostic exports are retained under `data/diagnostics` (or the non-portable application data directory).

Electron shell startup, renderer crashes, and Bun backend lifecycle events use the same structured log directory.

## Development

The app uses [Electron](https://www.electronjs.org/) for the desktop shell, [Bun](https://bun.sh/) for the local backend, Npcap on Windows, and libpcap or dumpcap on Linux for process-scoped packet capture.

### Prerequisites

- Windows x64 or Linux x64
- Bun 1.4 or newer
- Npcap on Windows; libpcap and preferably dumpcap on Linux

### Build from source

```sh
git clone https://github.com/bjb2/valemarket-desktop.git
cd valemarket-desktop
bun run setup
bun run check
bun run package:windows # on Windows
# or
bun run package:linux   # on Linux
```

Release artifacts are written under `dist/`. Build each platform on that platform so the packaged backend contains the correct Bun executable.

### Useful commands

```text
bun run dev              Prepare and launch a development build
bun run check            Run TypeScript checks and tests
bun run package:windows  Build the Windows x64 release archive
bun run package:linux    Build Linux x64 AppImage and tar.gz releases
```

## Project layout

```text
src/backend/      Platform capture lifecycle, local API, and contribution pipeline
src/electron/     Electron main process and restricted preload bridge
src/frontend/     Market browser and contributor controls
src/shared/       Shared desktop contracts
assets/           Local catalog, fonts, icons, and application artwork
test/             Contributor behavior tests
```

The public market API is operated separately at [market-api.spiritvalers.com](https://market.spiritvalers.com/api/). Its Cloudflare infrastructure, database, and deployment configuration are maintained in a private repository.
