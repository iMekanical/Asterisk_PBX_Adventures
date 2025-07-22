# Asterisk_PBX_Adventures 🚧📞  
## Real-world SIP configs, NAT nightmares, and full-conf redemption

Welcome to the trench notes of **Joseph Stacy**, where Asterisk PJSIP mysteries are hunted down, logged in context, and resolved with precision. This repo is a chronicle of hard-earned wins, built through relentless testing, exquisite data captures, and a strict demand:

> ✨ **Absolutely no piecemeal config advice. Full `.conf` files only. Every time.**  
> — Joseph Stacy

## 📘 Case 01: REGISTER Fails, 404 Not Found (AOR 10000)

**Symptoms:**
- MicroSIP softphone behind NAT.
- Asterisk returns `401 Unauthorized`, then `404 Not Found`.
- CLI warning: `AOR '' not found for endpoint '10000'`.

**The Setup:**
- Asterisk GIT master (2025)
- Public server IP: `172.235.32.192`
- MicroSIP client IP (behind eero NAT): `47.157.233.88`
- MicroSIP config was correct... almost.

**The Culprit(s):**
1. `contact_user=10000` was misconfigured (at one point in `[global]`, which is ignored by PJSIP).
2. A lowercase `auto` in MicroSIP's "Public Address" field poisoned the Contact header.
3. Auth section had a mismatched or stale password.
4. Asterisk’s AOR matching logic fails **silently** if the AOR doesn’t exist or isn’t bound properly.

**The Fix:**
- Removed "auto" from MicroSIP's Public Address field (left it empty).
- Ensured `[auth]`, `[aor]`, and `[endpoint]` sections matched — especially `aors=10000`.
- Corrected the password.
- Reloaded via `core reload`, `pjsip reload`, and `dialplan reload`.

## 🔍 CLI Logs, Full Confs, and Real Debugging

This repo includes:
- `pjsip.conf` and other Asterisk config files (entire sections, not snippets).
- Full PJSIP REGISTER attempts with `pjsip set logger on`.
- Call-ID tracking, Via headers, and nonce tracing.
- `pjsip show endpoint` and `pjsip show contacts` output.
- Companion Markdown breakdowns per case.

## 🤖 Co-Pilot Credit

Debugging co-piloted by [ChatGPT’s Asterisk Guru](https://docs.asterisk.org), who:
- Obeyed the full-conf-or-nothing rule.
- Analyzed SIP packet flows in context.
- Never suggested “have you tried turning it off and on again?”

## 🔧 Why This Repo Exists

Asterisk is powerful but *opaque* — especially when working with NAT, contact matching, and PJSIP endpoint identifiers. This repo serves as:
- A reference for known registration pitfalls.
- A structure for logging and resolving future edge cases.
- A growing archive of working, complete configurations.

## 💡 Pro Tips from This Case

- Leave MicroSIP’s "Public Address" field **blank** when behind NAT.
- AORs must be explicitly defined and referenced by endpoints.
- Always match your `auth`, `aor`, and `endpoint` names.
- Use `pjsip set logger on` early and often.
- `core reload` doesn't touch transport bindings; restart Asterisk when in doubt.

---

> _“If you don’t have `pjsip.conf` in front of you, you’re only guessing.”_  
> — J. Stacy

📎 See `case01-register-404-aor-10000.md` for the full trail of packet tears.

Stay tuned for more.
