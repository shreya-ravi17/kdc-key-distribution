# Symmetric Key Distribution with a KDC (Simulation)

A Python simulation of a **Key Distribution Center** that lets Alice obtain a
session key to talk securely with Bob, using the **Needham-Schroeder symmetric
protocol** (with ticket lifetime/expiry as in the Denning fix).

## Setup & run
```bash
pip install -r requirements.txt
python demo.py                      # full walkthrough + 7 scenarios
python -m unittest discover -s tests -v   # 13 unit tests
```

## Web frontend
Open `frontend/index.html` in Chrome, Edge or Firefox (just double-click). It runs the same protocol visually with real AES-GCM via the browser's Web Crypto API: step-by-step message flow, key panel, secure chat, and attack buttons (replay, tamper, eavesdrop, unregistered user, key expiry).

## Project layout
| File | Purpose |
|---|---|
| `kdc_sim/kdc.py` | KDC: registers principals, generates session keys, builds tickets |
| `kdc_sim/client.py` | Client (Alice/Bob): initiator + responder logic, secure messaging |
| `kdc_sim/crypto_utils.py` | AES-256-GCM encrypt/decrypt, key & nonce generation |
| `kdc_sim/network.py` | Simulated network that logs (and can tamper with) every message |
| `frontend/index.html` | Interactive visual simulation (no install needed) |
| `demo.py` | Demonstration of normal flow and attacks |
| `tests/test_kdc.py` | Unit tests |

## Protocol
Ka, Kb = master keys shared with the KDC (registered out-of-band).
Ks = fresh session key. N1, N2 = nonces.

```
1. A -> KDC : A, B, N1
2. KDC -> A : E(Ka, [Ks, B, N1, expires, Ticket])
              Ticket = E(Kb, [Ks, A, timestamp, expires])
3. A -> B   : Ticket
4. B -> A   : E(Ks, N2)
5. A -> B   : E(Ks, N2 - 1)
   then       E(Ks, message) in both directions
```

## Security properties
| Threat | Defence |
|---|---|
| Eavesdropping | All keys/messages are AES-GCM encrypted; master keys never sent |
| Impersonation to KDC | Only registered principals; reply decryptable only with Ka |
| Replay of KDC reply | N1 in reply must match Alice's request |
| Replay of ticket to Bob | Bob tracks seen tickets + expiry check |
| Old/compromised key reuse | Session key has a lifetime; new Ks per session |
| Tampering | GCM authentication tag fails -> message rejected |
| Ticket stolen by another user | Ticket binds identity A; Bob checks sender |
| Bob proves Alice has Ks | Steps 4-5 challenge-response |

## Limitations (discuss in report)
- KDC is a single point of failure and fully trusted.
- Master keys assumed distributed securely out-of-band.
- Simulation is in-process (no real sockets); `Network` models the channel.
- Clocks assumed loosely synchronised for expiry checks.
