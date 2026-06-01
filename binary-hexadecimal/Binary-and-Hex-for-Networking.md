# Binary and Hexadecimal for Networking

**Audience:** CCNA and networking students who need to read addresses, masks, and exam questions without fear of base-2 and base-16 math.

**Goal:** After this guide, you can convert between **decimal, binary, and hex** for one octet (0–255), use **powers of two** for subnetting, and recognize **why** networking uses hex at all.

---

## Part 1 — Why Networking Uses Binary and Hex

### 1.1 Computers speak in bits

A **bit** is one binary digit: `0` or `1`.

Networking addresses and masks are really **patterns of bits**. We write them in **decimal** (IPv4) or **hex** (IPv6, MAC addresses) because humans cannot scan 32 bits quickly.

| Human-friendly | What the machine uses |
| -------------- | --------------------- |
| `192.168.1.1`  | 32 bits grouped as 4 octets |
| `FF:AA:BB:CC:DD:EE` | 48 bits (MAC) |
| `2001:db8::1`  | 128 bits (IPv6) |

### 1.2 Why hex shows up so often

**Hexadecimal (base 16)** uses digits `0–9` and `A–F` (where A=10, B=11, … F=15).

One hex digit = **4 bits** (called a **nibble**).

| Bits | Hex |
| ---- | --- |
| 0000 | 0 |
| 0001 | 1 |
| 0010 | 2 |
| 0011 | 3 |
| 0100 | 4 |
| 0101 | 5 |
| 0110 | 6 |
| 0111 | 7 |
| 1000 | 8 |
| 1001 | 9 |
| 1010 | A |
| 1011 | B |
| 1100 | C |
| 1101 | D |
| 1110 | E |
| 1111 | F |

**Why CCNA cares:** IPv6 addresses are long in decimal, so they are written in hex. MAC addresses are always hex. Subnet masks and wildcard masks are easier to reason about in binary, then you write the result in decimal for configs.

---

## Part 2 — Powers of Two (Subnetting Fuel)

Memorize these values. Subnetting is mostly **which power of two** fits your host count or block size.

| Power | Value | Common use |
| ----- | ----- | ------------ |
| 2^0 | 1 | — |
| 2^1 | 2 | — |
| 2^2 | 4 | /30 has 4 addresses |
| 2^3 | 8 | /29 |
| 2^4 | 16 | /28 |
| 2^5 | 32 | /27 |
| 2^6 | 64 | /26 |
| 2^7 | 128 | /25 |
| 2^8 | 256 | /24 (one octet) |
| 2^9 | 512 | /23 |
| 2^10 | 1024 | /22 |
| 2^16 | 65536 | /16 |

**Host bits = h** -> usable hosts (typical LAN) = `2^h - 2`  
(Reserve network and broadcast addresses.)

**Prefix /24** -> 8 host bits -> `2^8 - 2 = 254` usable hosts.

---

## Part 3 — One IPv4 Octet (0–255)

An IPv4 address has **four octets**. Each octet is **8 bits**, so value range **0–255**.

Example: `192.168.10.75` — you often convert **one octet at a time**, not all 32 bits at once (unless an exam demands it).

### 3.1 Bit positions and weights

For one octet, bits are numbered **7 down to 0** (left to right):

```text
Bit position:  7   6   5   4   3   2   1   0
Weight:      128  64  32  16   8   4   2   1
```

**Value of octet** = sum of weights where bit = 1.

### 3.2 Decimal to binary

**Example: convert 192 to binary**

Which weights add up to 192?

- 128 fits -> 192 - 128 = 64 left
- 64 fits -> 64 - 64 = 0 done

```text
128 + 64 = 192  ->  11000000
```

**Example: convert 10 to binary**

```text
8 + 2 = 10  ->  00001010
```

**Example: convert 75 to binary**

```text
64 + 8 + 2 + 1 = 75  ->  01001011
```

### 3.3 Binary to decimal

**Example: `11000000`**

Add weights for each `1`:

```text
128 + 64 = 192
```

**Example: `11100000` (common in /27 masks)**

```text
128 + 64 + 32 = 224  ->  mask octet 255.255.255.224
```

### 3.4 Quick checks

- If the **leftmost bit** is 1 in the first octet, the decimal value is **128–255** (Classful-era trick; still useful for spotting patterns).
- **All 1s** in 8 bits = 255.
- **All 0s** = 0.

---

## Part 4 — Subnet Masks and CIDR in Binary

### 4.1 Prefix length = count of network bits

`/26` means **26 bits are 1** (network), **6 bits are 0** (host) in the 32-bit address.

Last octet for /26:

```text
11111111.11111111.11111111.11000000
                              ^^^^^^
                              6 host bits
```

Mask in decimal: `255.255.255.192` (last octet 128+64 = 192).

### 4.2 Finding the network address (one octet)

**Rule:** Network address = set all **host bits to 0** in that subnet.

**IP `10.50.120.75/26`** — last octet block size 64:

- Multiples of 64: 0, 64, 128, 192
- 75 is in 64–127 block
- Host bits in last octet for .75 with /26: network octet = **64**

You do not need full 32-bit binary on every exam question if you know **block sizes** (see [subnetting guide](../IP-addresses/subnetting.md)).

### 4.3 Wildcard masks (ACLs)

**Wildcard** = inverse of subnet mask bits (0 = must match, 1 = don't care in ACL logic).

Subnet mask octet `255` -> wildcard `0`  
Subnet mask octet `192` -> wildcard `63` (because 255 - 192 = 63)

Example ACL match `192.168.10.0 0.0.0.255` -> last octet "any."

---

## Part 5 — Hexadecimal for Networking

### 5.1 Hex to binary (4 bits at a time)

Split hex into nibbles.

**Example: `0xC4` or `C4`**

- C = 1100
- 4 = 0100
- Together: `11000100` = 196 decimal

**Example: MAC byte `AF`**

- A = 1010
- F = 1111
- `10101111`

### 5.2 Binary to hex

Group bits in **4s** from the right.

**Example: `11000000`**

```text
1100 | 0000  ->  C    0  ->  C0
```

### 5.3 Decimal to hex (via binary)

**Example: 255**

- Binary: `11111111`
- Hex: `FF`

**Example: 192**

- Binary: `11000000`
- Hex: `C0`

### 5.4 Practice table (memorize common mask octets)

| Decimal | Binary | Hex |
| ------- | ------ | --- |
| 255 | 11111111 | FF |
| 254 | 11111110 | FE |
| 252 | 11111100 | FC |
| 248 | 11111000 | F8 |
| 240 | 11110000 | F0 |
| 224 | 11100000 | E0 |
| 192 | 11000000 | C0 |
| 128 | 10000000 | 80 |
| 0 | 00000000 | 00 |

---

## Part 6 — MAC Addresses (48 Bits, Always Hex)

Format: **six octets**, often written `AA:BB:CC:DD:EE:FF`.

Example: `00:1A:2B:3C:4D:5E`

- First half often **OUI** (vendor); study **unicast vs multicast** (least significant bit of first octet).
- No subnet mask on a MAC — it is **Layer 2**.

**Convert one pair:** `1A` -> `00011010` -> 16+2 = **26** (only if exam asks decimal for one byte).

---

## Part 7 — IPv6 and Hex (CCNA Awareness)

IPv6 is **128 bits**, written as **8 groups of 4 hex digits**:

```text
2001:0db8:0000:0001:0000:0000:0000:0001
```

**Compression rules:**

1. Drop **leading zeros** in each group: `0db8` -> `db8`
2. Use **`::` once** for longest run of all-zero groups

Same address shortened:

```text
2001:db8:0:1::1
```

You rarely convert full IPv6 to binary by hand on CCNA — know **hex structure**, **::** rules, and **link-local** prefix `fe80::/10`.

---

## Part 8 — ANDing (Mask Applied to IP) — Classic Exam Skill

To find **network address**, you **AND** IP with subnet mask (bitwise):

- 1 AND 1 = 1  
- Anything AND 0 = 0  

**Example (last octet only):**

```text
IP:     75  ->  01001011
Mask:  192  ->  11000000
AND:         ->  01000000  = 64
```

Network ends in `.64` for that octet.

Do this per octet where mask is not 255.

---

## Part 9 — Speed Tricks for Exams

### 9.1 Last-octet increment (CIDR in 4th octet)

| Prefix | Host bits in last octet | Block size |
| ------ | ----------------------- | ---------- |
| /25 | 7 | 128 |
| /26 | 6 | 64 |
| /27 | 5 | 32 |
| /28 | 4 | 16 |
| /29 | 3 | 8 |
| /30 | 2 | 4 |

**Network octet** = largest multiple of block size not greater than IP octet value.

### 9.2 Magic number from host count

Need at least **50 hosts**:

- Try `2^5 - 2 = 30` (too small)
- Try `2^6 - 2 = 62` (fits) -> **6 host bits** -> prefix **/26**

### 9.3 Convert only the "interesting" octet

Mask `255.255.255.224` — only the last octet is not 255. Focus binary work there.

---

## Part 10 — Common Mistakes

| Mistake | Fix |
| ------- | --- |
| Confusing **hex digit** with **decimal** (C = 12, not "100") | Use nibble table |
| Forgetting **two** hex chars per byte | MAC has 12 hex chars = 6 bytes |
| Using **256** as a value in one octet | Max is **255** |
| Wrong bit weight order | Bit 7 is **128**, bit 0 is **1** |
| Applying /24 block math to **third octet** (/23, /22) | Block size moves left for smaller prefixes |

---

## Part 11 — Practice Problems

Try these, then check answers below.

### A. Decimal to binary

1. `255`  
2. `128`  
3. `240`  
4. `17`

### B. Binary to decimal

1. `10101000`  
2. `11110000`  
3. `00001111`

### C. Hex to decimal (one byte)

1. `FF`  
2. `A0`  
3. `2F`

### D. AND for network (last octet)

IP `192.168.5.87`, mask `255.255.255.192` — what is the network octet?

---

### Answers

**A.** 1) `11111111` 2) `10000000` 3) `11110000` 4) `00010001`  

**B.** 1) 168 2) 240 3) 15  

**C.** 1) 255 2) 160 (A=10 -> 160) 3) 47 (2=2, F=15 -> 32+15)  

**D.** 87 -> `01010111`, 192 -> `11000000`, AND -> `01000000` = **64** (network `192.168.5.64/26`)

---

## Part 12 — One-Week Drill Plan

| Day | Activity |
| --- | -------- |
| 1 | Memorize 4-bit hex table (0–F) |
| 2 | Convert 20 random decimals 0–255 to binary |
| 3 | Convert 20 binary strings (8 bits) to decimal |
| 4 | Memorize mask octet table (128–255) |
| 5 | 10 AND problems (last octet only) |
| 6 | Parse 5 MAC addresses into binary nibbles |
| 7 | Timed mixed quiz: 20 questions in 15 minutes |

---

## Related Files in This Repo

- [IP-addresses/subnetting.md](../IP-addresses/subnetting.md) — VLSM and CIDR design  
- [TCP_IP-model/TCP-IP-Model-Study-Guide.md](../TCP_IP-model/TCP-IP-Model-Study-Guide.md) — where PDUs and layers fit  
- [IP-addresses/IPv4_vs_IPv6_Study_Guide.md](../IP-addresses/IPv4_vs_IPv6_Study_Guide.md) — address formats  
- [cheat-sheets/command-cheat-sheet.md](../cheat-sheets/command-cheat-sheet.md) — IOS verification  

---

**CCNA takeaway:** You do not need to be a mathematician — you need **fast octet-level** binary/hex fluency and **powers of two** for masks. Practice until `224` instantly means `11100000` and `/27` means block size **32**.
