# Mozi Peer-to-Peer Botnet Architecture & Signatures

This dossier analyzes the Mozi P2P botnet, an IoT botnet that first emerged in late 2019 and dominated residential gateway compromise across Asia and India through 2021-2024.

---

## 1. Decentralized Peer-to-Peer Architecture

Unlike Mirai, which relies on centralized Command & Control (C2) servers (rendering them vulnerable to DNS sinkholing and IP takedowns), Mozi employs a **fully decentralized Peer-to-Peer (P2P) network** built on top of the public **BitTorrent Mainline Distributed Hash Table (DHT)**.

### Kademlia DHT Overlay Mechanics:
- Mozi nodes join the public BitTorrent DHT using standard Kademlia RPC messages (`ping`, `find_node`, `get_peers`, `announce_peer`).
- Mozi extends the Kademlia protocol by embedding custom payload fields in standard bencoded DHT messages.
- The DHT routing table maintains an 8-bucket tree of known live peers, allowing rapid multi-hop gossip routing.

```
       [Attacker / Master]
                |
                v  ECDSA384 Signed Payload
     +----------+----------+
     |                     |
     v                     v
 [Mozi Peer A] <-----> [Mozi Peer B]
     |                     |
     +----------+----------+
                |
                v
       [Mozi Peer C (CPE)]
```

---

## 2. Cryptographic Command Authentication

A defining characteristic of Mozi is its robust cryptographic command verification:
- **Asymmetric Signature Verification:** Every configuration update, executable payload download, or attack command must be signed using a **hardcoded ECDSA384 public key** embedded in the binary.
- **Rogue Takeover Prevention:** Security researchers and competing bot herders cannot hijack the botnet by injecting false DHT peers; without the attacker's private key, forged commands are rejected during signature verification.
- **Payload Verification Routine:** The binary hashes received DHT configuration blocks with SHA-256 and verifies the ECDSA curve signature prior to executing system shell calls.

---

## 3. Exploit Vectors & Vulnerable Target CPEs

Mozi propagates via automated exploitation of unpatched vulnerabilities in residential CPEs, GPON ONTs, and network cameras:
- **CVE-2017-17215:** Huawei HG532 remote code execution via UPnP SOAP interface (`/ctrlt/DeviceInfo`).
- **CVE-2018-10561 / CVE-2018-10562:** Dasan GPON home gateway remote code execution via path traversal (`/GponForm/diag_Form?images/`).
- **CVE-2020-10987:** Tenda AC9/AC11 router remote command execution via `goform/setSysTime`.
- **Realtek SDK Remote Code Execution (CVE-2014-8361):** Miniigd UPnP SOAP vulnerability widely present in OEM GPON ONTs (Syrotech, Netlink, Digisol) across Indian ISPs.
- **Netgear DGN1000 Setup.cgi:** Unauthenticated command execution.

---

## 4. Host Domination & "Vaccination" Routine

Upon successful exploitation:
1. Mozi downloads an architecture-specific ELF binary (`arm`, `mips`, `x86`) to `/tmp/` or `/var/tmp/`.
2. It renames itself to a random string or impersonates system daemons (e.g., `sshd`, `dropbear`).
3. It installs `iptables` rules blocking access to ports **23, 2323, 7547, 80, 8080** from external interfaces, effectively "vaccinating" the victim against competing botnets and preventing external remediation scans by the ISP.
4. It establishes persistence by modifying `/etc/init.d/`, `/etc/rc.local`, or appending to `crontab`.

---

## 5. Flow & Packet Signatures

Mozi-infected subscribers exhibit unmistakable network indicators:
1. **High UDP Session Churn on Ports 1024-65535:** The Kademlia DHT gossip engine generates continuous UDP `find_node` and `ping` queries to hundreds of distinct external IP addresses every minute.
2. **Bencoded String Payloads:** Flow inspection of UDP packets reveals BitTorrent DHT bencoding syntax: `d1:ad2:id20:...` containing Mozi's specific bot identifiers.
3. **HTTP User-Agent Strings:** Automated exploit downloaders frequently utilize hardcoded User-Agents such as `Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Trident/5.0)` or minimal curl/wget strings.
