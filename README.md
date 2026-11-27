# IT Support Portfolio

Hands-on IT support practice, written up the way a service desk would record it: as tickets
with a clear problem, steps taken, resolution and verification. It sits alongside my
[SOC home lab](https://github.com/alessandro-elemento/soc-home-lab), because service desk
work and security operations share the same habits of careful triage, documentation and
escalation.

Everything here is done on my own isolated lab machines. Where I break something on purpose
to practise fixing it, the ticket says so. Nothing is exaggerated, and each ticket records
what I did not test.

## Projects

| # | Project | Skills | Status |
|---|---|---|---|
| 05 | [Service desk tickets](templates/) | Ticket documentation, prioritisation, escalation, resolution | Template and priority matrix written |
| 01 | [Windows troubleshooting](01-windows-troubleshooting/) | Event Viewer, Task Manager, Windows services, application failures | 2 tickets done (IT-001 Print Spooler, IT-002 runaway process); 2 scenarios still planned |
| 02 | [Network troubleshooting](02-network-troubleshooting/) | DNS, DHCP, IP configuration, `ping`, `ipconfig`, `nslookup` | 1 ticket done (IT-003, DNS); 2 scenarios still planned |
| 04 | User onboarding | Windows user accounts, permissions, workstation setup | Not started |
| 03 | Microsoft 365 support | Outlook, Teams, OneDrive troubleshooting | Not started (needs a tenant to practise on) |

The numbers match my planning list, not the order I am doing them in. The ticket format
comes first because every other project is written up as a ticket.

## How each ticket is written

Every ticket uses [`templates/ticket-template.md`](templates/ticket-template.md) and the
priority rules in [`templates/priority-matrix.md`](templates/priority-matrix.md). A ticket
records:

- what the user reported, and what was actually wrong;
- each troubleshooting step in order, with the evidence for it;
- the root cause, the fix and how I verified the fix worked;
- whether it would need escalating and to whom;
- limitations: what I did not test or could not confirm.

## Repository structure

```
it-support-portfolio/
├── README.md
├── templates/
│   ├── ticket-template.md
│   └── priority-matrix.md
├── 01-windows-troubleshooting/
│   ├── README.md
│   └── tickets/
│       ├── IT-001-print-spooler-stopped.md
│       ├── IT-002-runaway-process-high-cpu.md
│       └── screenshots/IT-001/, IT-002/
└── 02-network-troubleshooting/
    ├── README.md
    └── tickets/
        ├── IT-003-websites-wont-load-dns-blocked.md
        └── screenshots/IT-003/
```
