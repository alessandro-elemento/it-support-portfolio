# Ticket: Websites will not load by name, DNS blocked

| Field | Value |
|---|---|
| Ticket ID | IT-003 |
| Opened | 27 November 2026, 11:20 AM (VM clock), when I started the first attempt. The fault that worked was in place from 11:34 AM. |
| Reported by | Self (deliberate lab fault) |
| Category | Network, DNS |
| Impact | Low (one user on one machine) |
| Urgency | High (assumed: every website that needs a name lookup fails) |
| Priority | P3 |
| Status | Resolved |
| Time to resolve | 12 minutes from the working fault (11:34 AM) to the verified fix (11:46 AM). The 14 minutes before that went on a first attempt that did not produce the fault. |

**Why this priority:** one user is affected, which is low impact. I rated urgency High because
no website loads by name, but there was no deadline and this is a lab, so that rating is an
assumption. It matches the priority matrix rule for one user with a clear problem
([priority matrix](../../templates/priority-matrix.md)).

## Lab disclosure

The fault was created on purpose on an isolated Windows 11 lab VM. I tried twice:

1. **Attempt 1 (11:20 AM):** pointed the network adapter at DNS server addresses that nothing
   should answer on. This did not stop name lookups from working, so I reverted it (see below).
2. **Attempt 2 (11:34 AM):** created two outbound Windows Firewall rules in an Administrator
   PowerShell window, one blocking UDP port 53 and one blocking TCP port 53, which is the port DNS
   uses. This produced the fault.

I recorded a baseline first (11:15 and 11:16 AM). Everything worked: `ping 8.8.8.8`, `nslookup
example.com`, `ping example.com`, and Google loaded in Edge.

## Summary

The user could reach the internet by address but not by name, because the computer was blocked
from asking a DNS server to turn names into addresses. Removing the firewall rules that blocked
DNS fixed it.

## What the user reported

"I can get online but websites won't load by name." (Written for the scenario. There was no real
user.)

## Environment

- Windows 11 on an isolated lab VM in VirtualBox (the same VM as IT-001 and IT-002), with the
  adapter shown as "UsbNcm Host Device" (Ethernet).
- Baseline from `ipconfig /all`: DHCP on, IPv4 `10.0.2.15` with mask `255.255.255.0`, default
  gateway and DHCP server `10.0.2.2`, and two DNS servers, `192.168.106.200` and
  `fd17:625c:f037:2::3`.
- I have left out the `ipconfig /all` screenshots, because they show the adapter's hardware (MAC)
  address. The values I used are quoted here instead.

## Attempt 1: a wrong DNS server address (did not produce the fault)

1. **Step:** At 11:20 AM I set DNS to Manual with IPv4 `192.0.2.1` (an address reserved for
   documentation, so nothing should answer on it) and left DNS over HTTPS off.
   **Result:** At 11:23 AM, after `ipconfig /flushdns`, `nslookup wikipedia.org` still got an answer.
   The server line read `192.0.2.1`.
2. **Step:** `ipconfig /all` at 11:23 AM.
   **Result:** DNS Servers listed `192.0.2.1` and `fd17:625c:f037:2::3`, the original IPv6 server.
   My first guess was that Windows was falling back to that second server.
3. **Step:** At 11:25 AM I asked `192.0.2.1` directly: `nslookup wikipedia.org 192.0.2.1`.
   **Result:** It still answered. That disproved the fallback guess, because a direct question to
   that address should have timed out.
4. **Step:** I also set the IPv6 DNS server to `2001:db8::1` (another documentation-only address).
   `ipconfig /all` at 11:27 AM listed only `2001:db8::1` and `192.0.2.1`. At 11:30 AM I tested again.
   **Result:** `nslookup bbc.co.uk` timed out against `2001:db8::1` ("DNS request timed out",
   timeout 2 seconds, then "Request to UnKnown timed-out"). But `ping bbc.co.uk` worked (resolved to
   `151.101.0.81`, 4 of 4 replies), `ping 8.8.8.8` worked, and `nslookup bbc.co.uk 203.0.113.5` (a
   third address nothing should answer on) also returned an answer.
5. **Step:** Set DNS back to Automatic (DHCP) at about 11:32 AM.
   **Result:** Settings showed Automatic (DHCP), and `ipconfig /all` listed `192.168.106.200` again.

**What I conclude:** on this VM, a wrong DNS server address does not break name lookups, because
queries to addresses where nothing should be listening still got answers. I do not know why. One
possible explanation is that the VM's virtual network answers IPv4 DNS queries itself, but I did
not confirm that. `nslookup` and `ping` also behaved differently, which I did not investigate:
`nslookup` used only the first server and timed out, while `ping` still resolved names.

## Troubleshooting steps (attempt 2: DNS blocked by firewall rules)

1. **Step:** At 11:34 AM I created the two blocking rules in an Administrator PowerShell window.
   **Result:** `LAB block DNS UDP` and `LAB block DNS TCP` were created, both Enabled, Outbound,
   Block, Profile Any.
2. **Step:** Tried a name I had not used before. At 11:35 AM I mistyped the flush command with a space
   (`ipconfig / flushdns`), which Windows rejected. I do not have a screenshot of a corrected flush
   before the tests below. At 11:39 AM I ran `nslookup abc.net.au`.
   **Result:** "DNS request timed out. timeout was 2 seconds." five times, then "*** Request to
   UnKnown timed-out". The server was `192.168.106.200`, my normal DNS server.
3. **Step:** `ping abc.net.au`.
   **Result:** "Ping request could not find host abc.net.au. Please check the name and try again."
4. **Step:** `ping 8.8.8.8`.
   **Result:** 4 of 4 replies, 15 to 20 ms. The connection itself works.
5. **Step:** `nslookup abc.net.au 8.8.8.8`, asking a different DNS server.
   **Result:** Timed out in the same way. So it is not one bad server. Port 53 is blocked from this
   machine.
6. **Step:** `Test-NetConnection 8.8.8.8 -Port 53`.
   **Result:** "WARNING: TCP connect to (8.8.8.8 : 53) failed", PingSucceeded True (28 ms),
   TcpTestSucceeded False.
7. **Step:** `Test-NetConnection 8.8.8.8 -Port 443`, as a comparison.
   **Result:** Also failed (TcpTestSucceeded False, PingSucceeded True at 15 ms). I did not expect
   that, and my rules do not block port 443. I had not tested this before the fault, so I could
   not tell whether it was normal.
8. **Step:** `Test-NetConnection 104.20.23.154 -Port 443` at 11:42 AM (the address `example.com`
   resolved to in my baseline).
   **Result:** TcpTestSucceeded True. So general internet access works, and `8.8.8.8:443` is the odd
   one. I do not know why it fails.
9. **Step:** `Get-NetFirewallRule -DisplayName "LAB*" | Format-Table DisplayName,Enabled,Direction,Action`.
   **Result:** Two rules, `LAB block DNS UDP` and `LAB block DNS TCP`, both True, Outbound, Block. (I
   created them, so I knew to look for them. On a real ticket I would list all blocking rules.)
10. **Step:** Opened `abc.net.au` in Edge at 11:43 AM.
    **Result:** "Hmmm... can't reach this page. abc.net.au's DNS address could not be found...
    diagnosing the problem now." with the code `DNS_PROBE_STARTED`. The page was still
    diagnosing, so the final error code may have changed.
11. **Step:** Looked at the network icon in the system tray.
    **Result:** At 11:41 AM I saw the tooltip "Network: No internet access". I took it with a phone
    photo of my screen, because I could not screenshot a tooltip. I have not included the photo.

## Root cause

Two outbound Windows Firewall rules blocked UDP and TCP port 53, so the computer could not send DNS
lookups. Names could not be turned into addresses, while traffic sent straight to an address kept
working. I know the cause because I created the rules, and the evidence fits: lookups failed whether
sent to my usual DNS server or to `8.8.8.8`, `ping 8.8.8.8` worked, a TCP test to port 53 failed
while port 443 to another address succeeded, the rules were listed as Block, and removing them fixed
it. I tested TCP port 53 only. I did not directly test UDP, which is what most DNS lookups use.

## Resolution

In the Administrator PowerShell window I ran `Remove-NetFirewallRule -DisplayName "LAB block DNS*"`
(screenshot at 11:44 AM), then `ipconfig /flushdns`. The command printed no output.

## Verification

At 11:46 AM:
- `ipconfig /flushdns` reported "Successfully flushed the DNS Resolver Cache."
- `nslookup abc.net.au` answered ("Non-authoritative answer", `23.221.132.116`) from `192.168.106.200`.
- `ping abc.net.au` resolved to `23.221.132.116` and got 4 of 4 replies (12 to 22 ms).
- `Get-NetFirewallRule -DisplayName "LAB*"` returned nothing.

The network icon in the system tray appears to have changed from a globe-style icon (at 11:44 AM) to
the normal network icon (at 11:46 AM). I did not capture the tooltip afterwards, so I cannot say
what it read. I did not retest the site in Edge after the fix.

Screenshots are in [`screenshots/IT-003/`](screenshots/IT-003/).

## Findings

- **A wrong DNS address did not break lookups.** Queries sent to `192.0.2.1` and `203.0.113.5`
  still got answers, though nothing real should answer there. A bad DNS setting is therefore not
  a reliable way to create this fault on this VM.
- **The tools disagreed.** With two dead DNS servers set, `nslookup` timed out while `ping` still
  resolved the name. On a real ticket I would test with more than one tool before deciding where
  the fault is.
- **Asking a second DNS server was a useful test.** `nslookup abc.net.au 8.8.8.8` timing out showed
  that the problem was not one DNS server.
- **Windows reported the problem too.** The "No internet access" tooltip appeared even though
  `ping 8.8.8.8` worked.

## Escalation

Not needed for this lab fault. On a real company machine I would not simply delete the rule. I would
first check whether it came from Group Policy or a security tool, because a firewall that blocks DNS
may be deliberate, for example to force DNS through an approved server. If it was managed centrally,
the rule would likely return, and I would take it to the network or security team with the rule name,
the test results above and the times. I did not test that scenario.

## Prevention and follow-up

- Keep a baseline of "what works" (for example `nslookup` and `ping` to a name and to an address)
  before changing anything, as I did here. I should also have tested `8.8.8.8:443` in the baseline.
- Possible follow-up: find out why `8.8.8.8:443` fails on this VM, and why IPv4 DNS to unreachable
  addresses is answered. I did not investigate either.

## Limitations

- I did not directly test UDP port 53, only TCP.
- I have no screenshot of a successful `ipconfig /flushdns` between the typo at 11:35 AM and the
  tests at 11:39 AM, so I cannot be sure the cache was flushed first. `abc.net.au` was a name I had
  not used before, which makes a cached answer unlikely.
- I did not check whether Edge was using its own secure DNS setting, and I did not retest Edge after
  the fix.
- The "No internet access" tooltip was seen once, at 11:41 AM, and is described in words only. The
  later change in the tray icon is not confirmed.
- I do not know why `nslookup` and `ping` differed in attempt 1, why `8.8.8.8:443` fails, or why DNS
  queries to unreachable IPv4 addresses were answered.
- The `ipconfig /all` screenshots are not in this repository because they show the adapter's MAC
  address.
- This is one fault on one VM whose virtual network may not behave like a real office network. Faults
  such as a wrong IP address, a failed DHCP lease or a wrong gateway were not tested here.
