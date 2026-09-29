# Troubleshooting Log — Lab 01

Faults introduced on purpose once each feature worked, then diagnosed as if they had been reported by a user. Aim for 5–8 entries.

**Fault ideas:** wrong access VLAN on a port · native VLAN mismatch on a trunk · missing `ip helper-address` · OSPF neighbor down (area or timer mismatch) · ACL line in the wrong order · NAT `inside`/`outside` swapped · port-security err-disabled port · wrong default gateway on a host

---

## Entry template

### #N — Short title

| | |
|---|---|
| **Symptom** | What the "user" reported / what failed |
| **Tested** | First checks (ping, `ipconfig`, etc.) |
| **Diagnosis** | Command(s) that revealed the cause, with the key output line |
| **Root cause** | One sentence |
| **Fix** | Exact config change |
| **Verification** | Command(s) proving it is fixed |
| **Takeaway** | What I would check first next time |

---
