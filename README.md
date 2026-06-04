# Task 4 — Firewall Setup and Configuration
Cybersecurity Internship | ElevateLabs

---

## What This Task Was About

This task was about setting up and using a firewall on Linux. The idea is to
understand how firewalls control what traffic is allowed into and out of a
machine. I used UFW (Uncomplicated Firewall) on Kali Linux which is basically
a simpler interface for managing Linux's built-in firewall (iptables).

The steps were — enable the firewall, block a specific port, test that the
block actually works, allow SSH, then clean up by removing the test rule.

---

## Setup

- **OS:** Kali Linux (WSL/Hyper-V on Windows)
- **Tool:** UFW (Uncomplicated Firewall)
- **Date:** June 3, 2026

---

## What I Did — Step by Step

### 1. Installed UFW

UFW wasn't installed by default on Kali so I had to install it first:

```
sudo apt install ufw -y
```

### 2. Enabled the Firewall

```
sudo ufw enable
sudo ufw status verbose
```

Output:

    Status: active
    Logging: on (low)
    Default: deny (incoming), allow (outgoing), disabled (routed)

So by default UFW blocks all incoming traffic and allows all outgoing. That's
actually a solid starting point — nothing gets in unless you explicitly allow it.

### 3. Blocked Port 23 (Telnet)

```
sudo ufw deny 23/tcp
```

Output:

    Rule added
    Rule added (v6)

Port 23 is Telnet. Telnet is an old remote access protocol from the 1960s that
sends everything — including passwords — in plain text with zero encryption.
Nobody should be running Telnet in 2026. Blocking it is just standard practice.

### 4. Allowed SSH (Port 22)

```
sudo ufw allow 22/tcp
```

SSH is the secure replacement for Telnet. It does the same thing (remote
terminal access) but with proper encryption. Allowing port 22 means I can
still SSH into the machine if needed.

### 5. Verified the Rules

```
sudo ufw status numbered
```

Output:

    Status: active

    To                         Action      From
    --                         ------      ----
    [ 1] 23/tcp                DENY IN     Anywhere
    [ 2] 22/tcp                ALLOW IN    Anywhere
    [ 3] 23/tcp (v6)           DENY IN     Anywhere (v6)
    [ 4] 22/tcp (v6)           ALLOW IN    Anywhere (v6)

UFW automatically added both IPv4 and IPv6 rules for each port which is good.

### 6. Tested the Block Rule

To actually verify the port 23 block was working I used netcat to try
connecting to it:

```
nc -zv 127.0.0.1 23
```

Output:

    localhost [127.0.0.1] 23 (telnet): Connection refused

Connection refused confirms the firewall is actively blocking port 23.
If the rule wasn't working it would either connect or just time out — getting
a refusal means the rule is doing its job.

### 7. Removed the Test Rule

After testing I removed the port 23 block rule to restore the original state
as the task required:

```
sudo ufw delete deny 23/tcp
```

Then verified:

```
sudo ufw status numbered
```

Output:

    Status: active

    To                         Action      From
    --                         ------      ----
    [ 1] 22/tcp                ALLOW IN    Anywhere
    [ 2] 22/tcp (v6)           ALLOW IN    Anywhere (v6)

Port 23 rule is gone. Only the SSH allow rule remains.

---

## All Commands Used

| Command | What it does |
|---|---|
| `sudo apt install ufw` | Install UFW |
| `sudo ufw enable` | Turn the firewall on |
| `sudo ufw status verbose` | Show firewall status and defaults |
| `sudo ufw status numbered` | List all rules with numbers |
| `sudo ufw deny 23/tcp` | Block incoming traffic on port 23 |
| `sudo ufw allow 22/tcp` | Allow incoming traffic on port 22 |
| `nc -zv 127.0.0.1 23` | Test if port 23 is reachable |
| `sudo ufw delete deny 23/tcp` | Remove the port 23 block rule |

---

## How a Firewall Filters Traffic

A firewall sits between incoming network traffic and your machine. Every
packet that arrives gets checked against the rules list from top to bottom.
The first rule that matches gets applied — if nothing matches, the default
policy kicks in.

In UFW the default policy here was deny incoming which means anything not
explicitly allowed gets dropped. This is called a whitelist approach and is
considered more secure than the alternative (allow everything except what's
blocked).

```
Incoming packet → Check rules top to bottom
                       ↓
              Rule matches? → Apply it (allow or deny)
              No match?     → Apply default policy (deny)
```

---

## Why Port 23 Specifically

The task uses port 23 as the example for good reason. Telnet was the standard
way to remotely access machines before SSH existed. The problem is it has no
encryption at all — every command you type and every response gets sent as
plain readable text across the network. Anyone sniffing the traffic can read
your username, password, and everything you do.

SSH (port 22) replaced Telnet in the late 1990s and encrypts everything.
There is basically no legitimate reason to have Telnet open on a modern system.

---

## Screenshots

Screenshots are in the repo:

- `ufw_rules.png` — status numbered output showing all 4 rules
- `port23_blocked.png` — netcat test showing Connection refused

---

## What I Learned

I didn't realise how straightforward UFW makes firewall management. The raw
iptables commands that UFW wraps around are significantly more complex —
UFW turns what would be a long iptables command into something like
`ufw deny 23/tcp` which is much easier to work with.

The netcat test was a good way to verify the rule actually worked rather than
just assuming it did. In security work you always want to verify that your
controls are actually doing what you think they are — a firewall rule that
looks right but doesn't work is worse than no rule because it gives false
confidence.

The default deny incoming policy is also something I want to remember — it's
a much safer starting point than allowing everything and trying to block the
bad stuff.

---

*ElevateLabs Cybersecurity Internship | Task 4 | June 2026*
