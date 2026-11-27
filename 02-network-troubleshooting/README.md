# 02 Network Troubleshooting

Practice diagnosing common network faults with the tools a service desk uses: `ipconfig`,
`ping`, `nslookup`, `Test-NetConnection` and the Windows network settings. I create each fault on
purpose in my isolated lab VM (Windows 11), diagnose it as if a user had reported it, fix it and write
it up as a ticket using the [ticket template](../templates/ticket-template.md).

**Status:** scenario A done (IT-003). Scenarios B and C are planned. Each ticket in
[`tickets/`](tickets/) should only be added once I have done the work and recorded what I
actually saw. Do not fill a ticket in from expectations.

## Scenarios

### Scenario A: websites will not load by name (DNS)

- **User report to write up:** "I can get online but websites won't load."
- **How I created the fault:** I first set the DNS server to an address nothing should answer on,
  but name lookups still worked on this VM (see IT-003). What worked was two outbound Windows
  Firewall rules blocking port 53 (UDP and TCP).
- **What to practise:** compare `ping` to an address and to a name, run `nslookup` against the
  default server and a different one, test port 53 with `Test-NetConnection`, and look for blocking
  rules.
- **Done:** [IT-003](tickets/IT-003-websites-wont-load-dns-blocked.md).

### Scenario B: wrong IP settings or a failed DHCP lease

- **User report to write up:** "I have no network at all."
- **Plan:** set a manual IPv4 address in the wrong subnet, or otherwise stop the VM getting a lease.
  Diagnose with `ipconfig /all` (is the address what I expect, is DHCP on) and by pinging the gateway.
  Fix by returning to Automatic (DHCP) and using `ipconfig /release` and `ipconfig /renew`.
- **Things to record:** the baseline address, gateway and lease times, and the exact messages.
- **Caution:** this will cut the VM's network while the fault is on, so expect Splunk and the log
  shipping to complain, and note it.

### Scenario C: finding where the connection breaks

- **User report to write up:** "The internet is down."
- **Plan:** use a deliberate fault (for example a firewall rule blocking outbound traffic) and work
  down a ladder: the machine's own address, the gateway, an outside address, then a name. Record what
  each result rules in or out.

## Checklist before each ticket is added

- [ ] The fault was created on the isolated lab VM, and the ticket's lab disclosure says so.
- [ ] A baseline was recorded before the fault, including anything I will use as a comparison.
- [ ] Every troubleshooting step has the real result I saw, not a guess.
- [ ] Commands and messages are copied from my screen.
- [ ] The fix was verified by repeating the original action.
- [ ] Limitations say what I did not test.
- [ ] Screenshots are checked for sensitive details, including the adapter's MAC address.

## Tickets

| Ticket | Scenario | Priority | Status |
|---|---|---|---|
| [IT-003](tickets/IT-003-websites-wont-load-dns-blocked.md) | A: DNS blocked, websites will not load by name | P3 | Resolved |

File names follow the pattern `IT-003-short-title.md`.
