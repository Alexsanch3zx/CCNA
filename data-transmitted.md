# Data Transmitted in Networking

## What Does "Transmitted" Mean?

In networking, **transmitted** means **sent from one device to another across a network**.

Examples:

- Your laptop sends a packet to a website → the packet is **transmitted**.
- A switch sends a frame out a port → the frame is **transmitted**.
- A router sends a packet toward its destination → the packet is **transmitted**.

Think of it like mailing a letter:

- **Transmit** = put the letter in the mail (send it)
- **Receive** = get the letter

---

## Networking Example

When you run:

```bash
ping google.com
```

Your computer:

1. Creates an ICMP packet.
2. **Transmits** the packet onto the network.
3. Routers forward it across the Internet.
4. Google's server receives it and sends a reply.
5. Your computer receives the reply.

---

## Common Terms

### TX (Transmit)
Sending data.

### RX (Receive)
Receiving data.

You'll often see this on Cisco devices:

```text
5 minute input rate 1000 bits/sec
5 minute output rate 2000 bits/sec
```

- **Input** = RX (received)
- **Output** = TX (transmitted)

---

## CCNA Tip

When a switch receives a frame on one port and sends it out another port:

- The frame is **received** on the ingress port.
- The frame is **transmitted** on the egress port.

Remember:

**Transmit = Send**
**Receive = Get**
