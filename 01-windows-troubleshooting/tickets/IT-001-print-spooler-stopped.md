# Ticket: Cannot print, Print Spooler service disabled

| Field | Value |
|---|---|
| Ticket ID | IT-001 |
| Opened | 26 November 2026, 4:22 PM (VM clock) |
| Reported by | Self (deliberate lab fault) |
| Category | Windows, services |
| Impact | Low (one user on one machine) |
| Urgency | High (assumed: the user cannot print at all) |
| Priority | P3 |
| Status | Resolved |
| Time to resolve | About 13 minutes from first symptom (4:22 PM) to verified fix (4:35 PM). This includes time spent capturing evidence, so a real ticket would likely be quicker. |

**Why this priority:** one user is affected, which is low impact. I rated urgency High because
printing is completely blocked, but no deadline was given and this is a lab, so that rating is
an assumption. The result matches the priority matrix example for "one user cannot print"
([priority matrix](../../templates/priority-matrix.md)).

## Lab disclosure

The fault was created on purpose on an isolated Windows 11 lab VM. Before creating it I
recorded a baseline: the Print Spooler service showed Status Running, and I printed a Notepad
document to Microsoft Print to PDF and saved it as `print-test-before.pdf` (4:20 PM). I then
opened the Services console, set the Print Spooler startup type to Disabled and stopped it
(about 4:21 PM). The Services console then showed the startup type as Disabled.

## Summary

The user could not print because the Print Spooler service, which Windows needs for any
printing, was disabled and not running. Setting it back to Automatic and starting it restored
printing.

## What the user reported

"I can't print anything." (Written for the scenario. There was no real user.)

## Environment

- Windows 11, build 10.0.26200.8037, on an isolated lab VM.
- Test application: Notepad, using the Microsoft Print to PDF printer. There is no physical
  or network printer on this VM.

## Troubleshooting steps

1. **Step:** Reproduced the problem by choosing File, Print in Notepad.
   **Result:** At 4:22 PM the print window did not work properly. The printer box was blank, the
   preview stayed on "Loading preview", and error code `0x800706ba` appeared.
2. **Step:** Looked up the error meaning. The code is the hexadecimal form of a Windows error
   number, so I ran `net helpmsg 1722` at 4:23 PM.
   **Result:** "The RPC server is unavailable." This points to a Windows service not answering,
   not to a printer or driver fault.
3. **Step:** Opened the Services console (`services.msc`) and found Print Spooler.
   **Result:** Status was blank (not running) and Startup Type was Disabled. The screenshot was
   taken at 4:21 PM, straight after I made the change.
4. **Step:** Opened Event Viewer, then Windows Logs, System, and looked at events from Service
   Control Manager around that time.
   **Result:** Two Event ID 7040 entries (a service start type was changed) for Print Spooler, at
   4:21:02 PM and 4:21:09 PM, with the User field `WINDOWS11-VICTI\kal`, the lab account. A third
   Event ID 7040 at 4:21:20 PM was for Windows Modules Installer, run as SYSTEM. It is a
   different service and I did not treat it as related.
5. **Step:** Looked for an Event ID 7036 entry showing Print Spooler entering the stopped state.
   **Result:** I did not find one. After the fix the list also showed no 7036 entry for the
   service starting, so the missing entry is not specific to the stop. I do not know why, so I have
   recorded it as a finding and not guessed a cause.

## Root cause

The Print Spooler service had its startup type set to Disabled and was not running. Printing
depends on that service, and the error `0x800706ba` ("The RPC server is unavailable") is
consistent with the print request having nothing to talk to. This is the best explanation, not
something I traced further. The Services console and the 7040 events both show the startup
type change, and the user field shows the lab account made it.

## Resolution

1. In the Services console, opened Print Spooler properties.
2. Changed Startup type from Disabled to Automatic and applied it.
3. Started the service.

Event Viewer recorded the change as Event ID 7040 at 4:33:40 PM, Source Service Control Manager,
User `WINDOWS11-VICTI\kal`: "The start type of the Print Spooler service was changed from
disabled to auto start." At 4:33 PM the Services console showed Print Spooler with Status
Running and Startup Type Automatic. The properties screenshot taken just before shows Startup type Automatic with the
service status still Stopped, which is the moment before Start was pressed.

## Verification

I repeated the original action. At 4:34 PM the Notepad print window showed Microsoft Print to PDF
selected in the printer box, and I saved the output as `print-test-after.pdf` at 4:35 PM. The
Documents folder then listed `print-test-before` (4:20 PM) and `print-test-after` (4:35 PM) side
by side, with the same size shown for both (the unit is cut off in the screenshot; the earlier
file was 32 KB).

Screenshots are in [`screenshots/IT-001/`](screenshots/IT-001/).

## Escalation

Not needed for this lab fault, it was resolved at first line. On a real company machine I would
check one thing before re-enabling the service: some organisations disable the Print Spooler on
purpose as a security measure. If Group Policy or a security team had done this, I would not
override it, and would ask them first. I have not tested that scenario.

## Prevention and follow-up

- Limit who can change service settings, and use Group Policy to keep Print Spooler on Automatic
  where printing is needed. I have not tested either.
- Possible follow-up: check the service Recovery tab to see what Windows does if the service
  fails. I did not look at it.

## Limitations

- Single deliberate fault on one VM, printing to a PDF printer only. Real printer drivers, print
  queues and network printers were not tested.
- No Event ID 7036 entry (stop or start) for Print Spooler appeared in the System log list I
  viewed, and the cause is unknown.
- The System log also showed Event ID 7040 entries at 4:29:07 PM, 4:32:51 PM and 4:37:24 PM. I
  did not open them, so I do not know which services they relate to.
- The baseline Startup Type text was cut off by the column width and began "Automatic (T". After
  the fix it reads "Automatic" with nothing after it. I do not know whether this means anything
  (it could be a trigger start setting, for example) and did not investigate.
- I did not open `print-test-after.pdf` to check its contents. Its existence, timestamp and size
  are the evidence.
- I did not restart the VM to confirm the service starts on its own at boot.
- The time to resolve includes time spent taking screenshots.
