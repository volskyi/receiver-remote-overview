# Receiver Remote: overview

> **Source code is private.** This repository describes the hardware link, the protocol and the software.

**A web remote for a Yamaha RX-V2700 AV receiver, driven over its RS-232C port: power, volume, mute,
input and DSP program from any phone, with live updates when someone uses the front panel or the IR remote.**
Plus a diagnostics toolkit for a Denon AVR-3808 over its network port.

![Python](https://img.shields.io/badge/Python-3-3776AB)
![pyserial](https://img.shields.io/badge/pyserial-RS--232C-informational)
![SSE](https://img.shields.io/badge/HTTP-Server--Sent%20Events-orange)

## Hardware link (Yamaha)

```
Mac / server ── USB–RS232 adapter ── null-modem ── receiver RS-232C (DE-9)
```

- **Connector:** DE-9 (DB-9) serial port on the receiver's back panel
- **Cable:** null-modem crossover, wired pin to pin: **2↔3** (RX/TX), **7↔8** (RTS/CTS), **5–5** (ground)
- **Line settings:** 9600 baud, 8N1
- **Gotcha:** the receiver only answers while **RTS is asserted**; the server holds it high
- The serial port can also be a network one (`socket://host:port` or `rfc2217://…`), so the receiver can sit
  behind ser2net or a serial-to-Ethernet bridge in another room

## Protocol and software

- Yamaha RS-232C protocol (RX-Vx700 series): commands as **STX … ETX** frames carrying remote-control
  codes; a DC1 status request answered by one DC2 frame with the full state
- The receiver **pushes report frames** when it is changed locally; the server parses them, and re-reads
  full status where a change is not reported back (power-on, speaker settings)
- Small HTTP server with a JSON API (`/api/power`, `/api/volume`, `/api/input`, `/api/program` …) and
  **Server-Sent Events**: every change, from any source, reaches every open browser
- Single-page mobile UI, no build step; protocol layer covered by unit tests

## Denon AVR-3808 diagnostics

Telnet control on port 23 (one session at a time). Used to map the setup menus that the web UI hides and to
localise a fault: test tones and pink noise, a microphone-based probe (tone on vs. muted, >10 dB = sound),
and a DLNA test server to separate analog paths from the DSP board. Result: analog bypass paths work,
everything through the DSP is silent, pointing at the digital/power board.

## My role

Wiring, protocol work, server and UI. Main themes: serial hardware control and hardware fault isolation by measurement.
