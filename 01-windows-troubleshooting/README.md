# 01 Windows Troubleshooting

Practice diagnosing common Windows faults using the tools a service desk uses: Event Viewer,
Task Manager and the Services console. I create each fault on purpose in my isolated lab VM
(Windows 11), diagnose it as if a user had reported it, fix it, and write it up as a ticket
using the [ticket template](../templates/ticket-template.md).

**Status:** scenarios planned. No tickets have been completed yet. Each ticket in
[`tickets/`](tickets/) should only be added once I have done the work and recorded what I
actually saw. Do not fill a ticket in from expectations.

## Scenarios

### Scenario A: a service has stopped

- **User report to write up:** "I can't print anything" (using the Print Spooler service).
- **How to create the fault:** open `services.msc`, stop the Print Spooler service, and set
  it to Disabled.
- **What to practise:** find the stopped service in the Services console, read its startup
  type, look in the Event Viewer System log for events from Service Control Manager around
  the time it stopped, restart it, set the startup type back, and verify by printing to PDF.
- **Things to record:** the exact event IDs and messages you see (do not rely on any
  numbers from memory), and the startup type before and after.

### Scenario B: a runaway process is making the computer slow

- **User report to write up:** "My computer has gone really slow and the fan is loud."
- **How to create the fault:** run a CPU-heavy loop in PowerShell, for example
  `while ($true) { }`, and leave it running.
- **What to practise:** open Task Manager, sort by CPU, identify the process and its path on
  the Details tab, check who started it, end the process, and confirm CPU returns to normal.
- **Things to record:** CPU percentage before and after, the process name and user.
- **Worth noting in the ticket:** how you would tell a harmless process from a suspicious one
  (path, publisher, parent process). This links to the SOC lab.

### Scenario C: slow start-up caused by start-up programs

- **User report to write up:** "It takes ages to be usable after I log in."
- **How to create the fault:** add a few programs to start at logon, using the Startup apps
  list in Settings or Task Manager.
- **What to practise:** use the Startup apps tab in Task Manager to see each item's start-up
  impact, disable the heavy ones, and compare boot-to-usable time before and after.
- **Things to record:** how you measured time, and be honest that a VM's timing varies, so
  repeat it a few times.

### Scenario D: an application error in Event Viewer

- **User report to write up:** "The program closed by itself."
- **How to create the fault:** choose a harmless way to make an application close
  unexpectedly in the lab, such as ending it from Task Manager while it is running. This may
  not generate the same events as a genuine crash, so say so in the ticket.
- **What to practise:** use Event Viewer's Application log and Reliability Monitor to look for
  an entry about the failure, and record what you did and did not find.

## Checklist before each ticket is added

- [ ] The fault was created on the isolated lab VM, and the ticket's lab disclosure says so.
- [ ] Every troubleshooting step has the real result I saw, not a guess.
- [ ] Event IDs and messages are copied from my screen.
- [ ] The fix was verified by repeating the original action.
- [ ] Limitations say what I did not test.
- [ ] Any screenshot is checked for sensitive details.

## Tickets

None yet. File names follow the pattern `IT-001-short-title.md`.
