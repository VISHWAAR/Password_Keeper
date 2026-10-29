# Password Keeper

A lightweight, offline password manager built in Python and shipped as a standalone Windows executable. Built in 2022 — before I knew the word "cybersecurity" applied to what I was doing.

---

## The Core Idea

Most password managers are cloud-based. Your passwords live on someone else's server, encrypted or not, and travel across a network. This project rejects that entirely.

**Everything stays on your machine. No internet. No server. No API. No cloud.**

The only thing that can read your stored passwords is the Python script itself — and the `.exe` compiled from it.

---

## The Shoulder Surfing Problem

The specific threat this was designed against is **shoulder surfing** — someone physically looking at your screen, or glancing at a file open on your desktop, and reading your password directly.

Most "secure" local storage still fails this test. A text file with your passwords is readable by anyone who opens it. Even a password-protected ZIP can be opened if someone sees you type the password.

This app solves it differently. Passwords are never stored as readable text anywhere on disk. If someone opens the storage file, here is what they see:

```
01110000 01100001 01110011 01110011 00100000 01101101 01100101 00100000 01110100 01101000 01100101 00100000 01110011 01100001 01101100 01110100
```

A shoulder surfer sees 1s and 0s. Nothing they can use.

---

## How the Storage Works

### Encoding (saving a password)

1. The user types a password into the app.
2. Each character is converted to its **binary representation** (a sequence of 1s and 0s).
3. The binary chunks are joined with a **custom delimiter** — a separator that would never appear naturally in the binary string.
4. The resulting string of 1s, 0s, and delimiters is written to a plain file on disk.

### Decoding (retrieving a password)

1. The app reads the file.
2. It **splits** the binary string on the delimiter, recovering the individual binary numbers.
3. Each binary number is converted to its **integer value**.
4. That integer is mapped back to a character through a custom algorithm.
5. The original password is reconstructed and shown only inside the app.

The file on disk is just text — but text that is meaningless without knowing the delimiter and the character-mapping algorithm. Both live only inside the Python script (and the compiled `.exe`). The storage format is the obfuscation.

---

## Why This Works Against Shoulder Surfing

| Scenario | Outcome |
|----------|---------|
| Someone looks over your shoulder at the app | They see `••••••••` or a UI — not the password |
| Someone opens the storage file in Notepad | They see binary garbage — no readable text |
| Someone copies the storage file | Useless without the `.exe` or the source script |
| Someone looks at your screen while the file is open | Still just 1s and 0s |

The attack surface for visual theft is effectively zero.

---

## Why Offline-Only Makes This Stronger

Cloud password managers have to solve hard problems: encrypted sync, server-side breaches, man-in-the-middle attacks, API key leaks, account takeovers. Each of those is an attack vector.

This app has none of them — because it never connects to anything.

| Attack Vector | Cloud Manager | This App |
|--------------|---------------|----------|
| Server breach | Possible | Not applicable |
| Network interception | Possible | Not applicable |
| API vulnerability | Possible | Not applicable |
| Phishing for master password | Possible | Not applicable |
| Physical access to machine | Possible | Possible |

The only real attack vector is physical access to the machine itself — and even then, the storage files require the executable to be useful.

---

## Distribution

The Python script is compiled into a standalone **Windows `.exe`** using the **Inno Setup Compiler**, producing a single installable binary. This means:

- No Python installation required on the end machine.
- The source code and the delimiter/mapping logic are bundled and not casually inspectable.
- The app behaves like any other installed Windows software.

---

## What I Didn't Know I Was Doing (2022)

At the time, this was just a project. Looking back, it touched several real concepts in information security:

**Obfuscation as a defense layer** — storing data in a non-human-readable format to defeat casual inspection. Not encryption in the cryptographic sense, but a meaningful barrier against the specific threat (visual theft) it was designed to stop.

**Threat modelling** — even without knowing the term, the design was shaped around a specific attacker (a shoulder surfer) and a specific attack (visual password theft). The solution was proportionate to that threat.

**Air-gapping** — keeping sensitive data completely off any network is a real security principle used in high-security environments. This app applies it at the personal level.

**Security through locality** — the algorithm that decodes the passwords exists only in one place: the script. There is no key server, no recovery mechanism, no external dependency. Possession of the `.exe` is the key.

**Binary encoding as lightweight obfuscation** — while not cryptographically secure, the encoding is genuinely effective against the targeted attack. A cryptographer would not be impressed, but a shoulder surfer would learn nothing.

This was an accidental, intuitive entry into cybersecurity thinking — solving a real problem with a real (if simple) security model, before knowing any of the formal vocabulary for it.

---

## Limitations and Honest Caveats

- **Not cryptographic encryption.** The encoding is obfuscation, not AES or RSA. Someone who knows the scheme (binary encoding with a delimiter + integer-to-character map) could reverse it without the app. Security depends on the attacker not knowing the algorithm.
- **No master password.** Anyone who can run the `.exe` on your machine can access the passwords. Physical machine security matters.
- **No breach detection.** If the storage files are tampered with, the app has no way to detect it.

For the threat it was designed to stop — someone glancing at your screen or your files — it works. For nation-state-level adversaries, use something else.

---

## Tech Stack

| Component | Technology |
|-----------|-----------|
| Core logic | Python |
| File storage | Local filesystem (plain files) |
| Encoding | Binary + custom delimiter + character mapping |
| Distribution | Inno Setup Compiler → `.exe` |
| Network usage | None |

---

*Built in 2022. My first step into security — without knowing that's what it was.*
