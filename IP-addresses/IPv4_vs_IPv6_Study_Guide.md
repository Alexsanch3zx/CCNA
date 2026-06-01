# IPv4 vs IPv6

## Comparison Table

| Feature | IPv4 | IPv6 |
|----------|--------|--------|
| Address Length | 32-bit | 128-bit |
| Example Address | `192.168.1.10` | `2001:0db8:85a3::8a2e:0370:7334` |
| Total Addresses | ~4.3 billion | ~340 undecillion (3.4 × 10^38) |
| Format | Decimal | Hexadecimal |
| Address Sections | 4 octets | 8 groups |
| NAT Needed? | Often yes | Usually no |
| Configuration | Manual or DHCP | SLAAC, DHCPv6, Manual |
| Security | Optional IPsec | Built-in IPsec support |
| Broadcasts | Uses broadcasts | No broadcasts (uses multicast) |

## IPv4

Example:

`192.168.1.100`

Each section (octet) is 8 bits.

Binary:

`11000000.10101000.00000001.01100100`

Total:

`4 octets × 8 bits = 32 bits`

Maximum addresses:

`2^32 = 4,294,967,296`

## IPv6

Example:

`2001:0db8:85a3:0000:0000:8a2e:0370:7334`

Each section is 16 bits.

Total:

`8 groups × 16 bits = 128 bits`

Maximum addresses:

`2^128`

Which is:

`340,282,366,920,938,463,463,374,607,431,768,211,456`

## Shortened IPv6 Addresses

Full:

`2001:0db8:0000:0000:0000:0000:1428:57ab`

Shortened:

`2001:db8::1428:57ab`

Rules:

1. Remove leading zeros.
2. Replace one continuous group of zeros with `::`.

## Why IPv6 Exists

The internet kept growing:

- Computers
- Phones
- Tablets
- Smart TVs
- IoT devices

IPv4's address space became insufficient, so IPv6 was created.

## CCNA Exam Facts

### IPv4
- 32-bit address
- Uses decimal numbers
- Example: `10.0.0.1`
- Subnetting is heavily tested

### IPv6
- 128-bit address
- Uses hexadecimal
- Example: `2001:db8::1`
- No broadcast addresses
- Uses multicast and anycast
- Often written with compressed notation

## Quick Memory Trick

**IPv4 = 4 sections separated by dots**

`192.168.1.1`

**IPv6 = 8 sections separated by colons**

`2001:db8::1`

Think:

> IPv4 = small address space (32 bits)  
> IPv6 = huge address space (128 bits)
