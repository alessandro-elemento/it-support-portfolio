# Ticket: Computer very slow, PowerShell process using CPU

| Field | Value |
|---|---|
| Ticket ID | IT-002 |
| Opened | 26 November 2026, 4:47 PM (VM clock), when the first attempt started |
| Reported by | Self (deliberate lab fault) |
| Category | Windows, performance |
| Impact | Low (one user on one machine) |
| Urgency | High (in the first attempt the machine froze completely) |
| Priority | P3 |
| Status | Resolved |
| Time to resolve | About 13 minutes from the first loop (4:47 PM) to CPU back down (about 5:00 PM), including a VM reset. The second attempt took about 2 to 3 minutes from starting the loop to ending it. |

**Why this priority:** one user is affected, which is low impact. If the computer had only been
slow, I would have rated urgency Medium, which gives P4. I rated it High and P3 because in the
first attempt the VM stopped responding completely and I could not even open Task Manager. This
follows the rule in the [priority matrix](../../templates/priority-matrix.md) to re-rate a
ticket when you learn more.

## Lab disclosure

The fault was created on purpose on an isolated Windows 11 lab VM, by running a CPU-heavy loop in
PowerShell. I tried it twice:

1. **Attempt 1 (4:47 PM):** `while ($true) { }`, which never stops. The VM froze, and it had to be
   restarted.
2. **Attempt 2 (about 4:58 PM):** `$end = (Get-Date).AddSeconds(180); while ((Get-Date) -lt $end) { }`,
   a loop that stops itself after 3 minutes, so it could not leave the VM stuck again.

The PowerShell prompt showed `C:\WINDOWS\system32` in both attempts, which is where an
administrator session normally opens. I did not check, so I think both were elevated but I am
not certain.

## Summary

A PowerShell process was using one processor's worth of CPU, which would make a real computer
slow and noisy. I found it in Task Manager and ended it, and CPU returned to normal. The first
attempt also froze the whole VM, which is recorded below as a finding.

## What the user reported

"My computer has gone really slow and the fan is loud." (Written for the scenario. There was no
real user.)

## Environment

- Windows 11, build 10.0.26200.8037 (recorded in [IT-001](IT-001-print-spooler-stopped.md)), on an
  isolated lab VM running in VirtualBox.
- Most processes on Task Manager's Details tab show Arm64 (`powershell.exe` does), so the guest looks
  like an Arm VM. One Splunk process shows x64.
- 4 logical processors, 4 GB of memory (72 to 79% used in every screenshot), virtual hard disk.
- The VM is not idle by default. It runs Splunk (`splunkd`), MongoDB, Microsoft Defender and
  Windows Update, and it feels slow even before the fault.

## Troubleshooting steps

1. **Step:** Took a baseline in Task Manager (attempt 1, 4:45 and 4:46 PM).
   **Result:** Overall CPU was 52%, then 67%. The top process was Windows Modules Installer Worker
   at 16.0%, then Service Host: Windows Update at 5.7%. Memory was 79%. Windows Update looks like
   the cause from the process names, but I did not confirm it. This was not a quiet baseline.
2. **Step:** Started the unbounded loop (attempt 1, 4:47 PM).
   **Result:** The screen froze immediately after I started the loop. I reported it at 4:49 PM and
   it was still completely frozen at 4:51 PM, so I could not open Task Manager. The VM was then
   restarted from VirtualBox (I did not record how). Task Manager's uptime of 8 minutes 34 seconds
   at 5:00 PM puts the restart at about 4:52 PM, and the desktop was usable by 4:55 PM. I do not know what caused the
   freeze. It began after I started the loop, and I did not establish that the loop was the cause.
3. **Step:** Took a baseline after the reboot (4:55 and 4:56 PM).
   **Result:** Overall CPU 51% at 4:55 PM, then 21% at 4:56 PM, with memory at 72% then 78%. At 4:56 PM
   the top process was `splunkd service` at 9.9%, then Task Manager at 3.1%.
4. **Step:** Typed the timed loop at 4:57 PM and pressed Enter.
   **Result:** At 4:57 PM overall CPU was still 7 to 8%, meaning the loop had not started. At
   4:58 PM it was 48%.
5. **Step:** Sorted the Processes tab by CPU at 4:59 PM.
   **Result:** Two screenshots 16 seconds apart. At 4:59:05 PM overall CPU was 68%. `splunkd service`
   was top at 35.8% (disk 19.7 MB/s) and Windows PowerShell second at 19.0%. At 4:59:21 PM overall
   CPU was 28%, Windows PowerShell was top at 23.7%, and `splunkd service` was down to 0.8%.
6. **Step:** Opened the Details tab at 5:00 PM.
   **Result:** `powershell.exe`, PID **4944**, User name **kal**, CPU 24, memory 26,412 K, status
   Running, description "Windows PowerShell".

## Root cause

A `powershell.exe` process (PID 4944, run by the lab account `kal`) was stuck in a tight loop,
using about 24% of overall CPU, which is roughly one of the four logical processors. I know this
because I started it, and it matches Task Manager: it was the top process at 23.7% in one
screenshot, and CPU fell once it was ended. The cause of the first attempt's freeze is not known.

## Resolution

Selected `powershell.exe` (PID 4944) on the Details tab and used **End task**. The Details
screenshot was taken at 5:00:21 PM and the screenshot after at 5:00:55 PM, so I ended it between
those times. I did not record the exact second.

## Verification

At about 5:00 PM Task Manager's Performance tab showed overall CPU at 5% and falling, 133 processes
(139 at 4:58 PM), and the PowerShell terminal icon had gone from the taskbar. This is one reading
with the graph still sloping down, not an average over time.

Screenshots are in [`screenshots/IT-002/`](screenshots/IT-002/).

## Findings

- **The busiest process was not always the loop.** At 4:59:05 PM `splunkd service` used more CPU
  (35.8%) than PowerShell (19.0%), and 16 seconds later it had dropped to 0.8%. On a real ticket,
  one reading is not enough. I would check a few readings and ask what is expected to be busy.
- **An unbounded loop froze the VM.** A loop with a time limit worked without a freeze. This VM is
  small (4 GB, 4 processors), and it is already working hard before any fault is added.

## Escalation

Not needed for this lab fault. On a real machine I would not just end the process. First I would
check where it runs from, who started it, and what its command line is, because a user-run PowerShell
process using a lot of CPU could be harmless or could be malicious. If I could not explain it, I would
escalate to the security team whatever the priority score says, as in the priority matrix. I did not
test any of those checks here.

## Prevention and follow-up

- Give the VM more memory or processors, or avoid running Splunk during tests. I have not tried either.
- Use a time-limited loop for future CPU tests.

## Limitations

- The cause of the freeze in attempt 1 is unknown, and I did not establish that the loop caused it.
- I did not record how the VM was restarted (for example Reset or Power off). The restart time
  (about 4:52 PM) is worked out from Task Manager's uptime.
- Both baselines were busy (Windows Update in attempt 1, Splunk in attempt 2), so before and after
  CPU numbers are rough.
- I did not capture the **Command line** column or the file location of the process, so I cannot
  say how it was launched. I also did not check its parent process. System Informer is installed on
  the VM and would show that, but I did not use it.
- Whether the sessions were elevated is inferred from the prompt path, not confirmed.
- The exact time I ended the process is only known to within the screenshot times (5:00:21 to 5:00:55 PM).
- The drop from 139 to 133 processes is more than the one process I ended, and I do not know why.
- The CPU after reading is a single point on a falling graph.
- The loop was written in PowerShell on an Arm64 VM. Other kinds of runaway processes, such as a
  memory leak or a process that keeps restarting, were not tested.
