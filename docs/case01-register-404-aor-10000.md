# Case 01: REGISTER Fails with `404 Not Found`  
## AOR 10000 | MicroSIP → Asterisk (PJSIP) | NAT | Misnamed AOR

> ⚠️ **Disclaimer:** All IP addresses and domain names below are anonymized and non-routable. Any resemblance to real-world infrastructure is coincidental.

---

### 🧠 TL;DR

- **Client:** MicroSIP 3.21.6 behind NAT.
- **Server:** Asterisk (GIT-master) on a public IPv4.
- **Symptom:** Softphone sends REGISTER. Auth works. Server replies `404 Not Found`.
- **Cause:** Endpoint was defined, but the AOR name was mismatched or misreferenced.
- **Fix:** Unified AOR name (`10000`) across `[endpoint]`, `[auth]`, and `[aor]`.

---

### 🖥️ Network Overview

| Device             | IP               | Role        |
|--------------------|------------------|-------------|
| Asterisk           | 198.51.100.2     | Public PBX  |
| MicroSIP           | 203.0.113.88     | Home NAT    |
| MicroSIP Internal  | 192.0.2.30       | LAN Client  |

*All addresses are [RFC 5737](https://datatracker.ietf.org/doc/html/rfc5737) test-net ranges.*

---

### 📜 Original PJSIP Registration Attempt


REGISTER sip:pbx.example.net SIP/2.0
Contact: <sip:10000@192.0.2.30:59759;ob>
From: "Joseph Stacy <+14157992242>" <sip:10000@pbx.example.net>
Authorization: Digest username="10000", realm="asterisk", ...
Asterisk Response:

SIP/2.0 404 Not Found
WARNING[xxxxx]: res_pjsip_registrar.c:1239 find_registrar_aor:
  AOR '' not found for endpoint '10000' (203.0.113.88:59759)
🔍 Diagnosis
CLI Tools Used

pjsip show endpoint 10000
pjsip show aors
pjsip show auths
pjsip show identifies
pjsip set logger on
Observations
Authentication succeeded (200 OK after digest).

But Asterisk couldn't match to any AOR → 404 returned.

The [aor] section had a different name than the aors= entry in [endpoint].

Also, a lingering contact_user=10000 in [global] was ignored by PJSIP and may have confused expectations.

✅ Fix Summary
pjsip.conf (Corrected & Working)
[transport-udp]
type=transport
protocol=udp
bind=0.0.0.0:5060
local_net=192.0.2.0/24
external_signaling_address=198.51.100.2
external_media_address=198.51.100.2

[10000]
type=endpoint
aors=10000
auth=10000
context=from-internal
disallow=all
allow=ulaw,alaw,opus
direct_media=no
rewrite_contact=yes
rtp_symmetric=yes
force_rport=yes

[10000]
type=aor
max_contacts=3

[10000]
type=auth
auth_type=userpass
username=10000
password=superSecret123

[10000_identify]
type=identify
endpoint=10000
match=203.0.113.88
MicroSIP Settings
Username: 10000

Domain: pbx.example.net

Outbound Proxy: left blank

Public Address: cleared (was “auto”)

🛰️ CLI Output After Fix
text
Copy
Edit
Endpoint:  10000     Unavailable
InAuth:    10000/10000
AOR:       10000     3 max contacts
Contact:   sip:10000@192.0.2.30:59759;ob → ADDED
✔️ Contact appears in pjsip show contacts
✔️ No more 404s

🧬 What We Learned
Asterisk doesn’t throw obvious errors when aors= references an undefined [aor] section.

PJSIP silently fails registration if aor can't be resolved — even after successful auth.

The contact_user global setting is deprecated or irrelevant under PJSIP.

pjsip reload isn’t always enough; core reload or even a full restart may be needed to flush stale object state.

NAT traversal requires rtp_symmetric, rewrite_contact, and force_rport.

👥 Credits
Joseph Stacy – Provided laser-focused test data, insisted on full .conf visibility, and showed exceptional patience.

ChatGPT Asterisk Guru – Deciphered registrar errors, read PCAPs, and narrated the resolution process.

