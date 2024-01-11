---
layout: post
title:  "Setting Processor Affinities on Windows"
math: false
categories: learning
---


Setting processor affinities for processes on Windows can be complex. This guide explains different approaches, their limitations, and when to use each one. 

## Background: Processor groups, Affinity APIs and CPU Sets APIs

To support systems with more than 64 logical processors, Windows 7 and Windows Server 2008 R2 introduced _processor groups_ - sets of up to 64 logical processors each. The system assigns processors to groups in a way that minimizes the total number of groups while maximizing physical proximity (e.g., keeping logical processors from the same physical core or NUMA node in the same group when possible).

Before Windows 11 and Server 2022, processes were constrained to a single processor group. If you started a process on group 0, it couldn't create threads on group 1. Process and thread affinities were governed by **Affinity APIs**, which still work but retain vestiges of this single-group limitation.

Windows 11 and Server 2022 introduced **CPU Sets APIs** that allow processes and threads to span multiple processor groups. However, these introduce new limitations: child processes don't inherit CPU Sets settings (though you can work around this by fully managing process creation/termination).

We can work around Affinity APIs single-group limitation by manually setting individual threads' affinities with `SetThreadGroupAffinity` and `SetThreadAffinityMask`, and we can work around CPU Sets APIs inheritance limitations by fully managing child process creation/termination. Both these "workarounds" are probably not feasible for the majority of programs.

## Solutions

### 1. (Low-hanging fruit) Using the `start` command 

While it has some limitations, this is an **excellent and simple solution for most single-group scenarios**. If your use-case allows it, I recommend going with this.

{%highlight cmd%}
start /node <numa-node> /affinity <affinity-mask> <executable>
{%endhighlight%}

Pros:

- Extremely easy to do.
- Child processes inherit parents' affinities.
- Possible to set affinities on creation.
- You're not constrained to any processor group.

Cons:

- Impossible to set multi-group affinities.
- A bit hacky, because NUMA nodes may not always translate to processor groups.

The affinity mask is a hexadecimal number where each bit represents a processor. For example, `0xC00000` (or just `C00000` in the command) sets bits 22 and 23 to 1 (binary: `1100 0000 0000 0000 0000 0000`), allowing the process to run only on processors 22 and 23.

### 2. (Also low-hanging fruit) Using PowerShell to start processes and then modifying their processor affinities

This solution is mentioned for completeness, as its solution space is likely a subset of what can be achieved with the `start` command. Both are simple approaches, but `start` has fewer limitations. 

{%highlight txt%}
$process = Start-Process <executable> -PassThru
$process.ProcessorAffinity = <affinity-mask>
{%endhighlight%}

Pros: 

- Extremely easy to do.
- Child processes inherit parents' affinities.

Cons:

- Impossible to set affinities on creation. 
	- There is no guarantee that threads created before the affinity is set will run on the processors you've chosen.
- Impossible to set multi-group affinities.
- You're constrained to the processor group PowerShell was started in.

### 3. Using CPU Sets APIs

If you're going to use Windows APIs, you likely can manage the full lifecycle of process creation/termination. If you can do that, **CPU Sets APIs are preferable to Affinity APIs when you need multi-group affinity support.** They're easier to manage since you don't need to set individual thread affinities, though you do need to set affinities for every child process if you want them to inherit their parent's settings — it's not automatic.

While Affinity APIs provide a straightforward way to set process affinities upon creation, CPU Sets APIs don't. You can work around this limitation by using process creation functions (see `CreateProcess` on [6]) with a `CREATE_SUSPENDED` flag. With the primary thread suspended, you can set CPU set affinities using `SetProcessDefaultCpuSets` or `SetThreadSelectedCpuSets`, then resume the thread with `ResumeThread`.

[Here's](https://learn.microsoft.com/en-us/windows/win32/procthread/cpu-set) the link to CPU Sets APIs (also included in reference [2]).

Pros:

- Possible to set affinities on creation.
- **Possible to set multi-group affinities in an easy way (without managing individual thread affinities, as would be the case with Affinity APIs).**

Cons:

- **Child processes do not inherit parents' CPU set affinities.**
	- You need to fully manage processes' lifecycles if you want to have children processes inheriting parents' CPU set affinities.

### 4. Using Affinity APIs

As mentioned before, Affinity APIs still have vestiges of their single-group limitation and require extra work to support multi-group processes. However, children automatically inherit their parent's affinities.

To set affinities on creation and to manage multi-group processes with affinity APIs, follow guidelines on [1]:

> A thread's affinity can be specified at creation using the [PROC_THREAD_ATTRIBUTE_GROUP_AFFINITY](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-updateprocthreadattribute) extended attribute with the [CreateRemoteThreadEx](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createremotethreadex) function. After the thread is created, its affinity can be changed by calling [SetThreadAffinityMask](https://learn.microsoft.com/en-us/windows/desktop/api/WinBase/nf-winbase-setthreadaffinitymask) or [SetThreadGroupAffinity](https://learn.microsoft.com/en-us/windows/win32/api/processtopologyapi/nf-processtopologyapi-setthreadgroupaffinity). If a thread is assigned to a different group than the process, the process's affinity is updated to include the thread's affinity and the process becomes a multi-group process. Further affinity changes must be made for individual threads; a multi-group process's affinity cannot be modified using [SetProcessAffinityMask](https://learn.microsoft.com/en-us/windows/desktop/api/WinBase/nf-winbase-setprocessaffinitymask). The [GetProcessGroupAffinity](https://learn.microsoft.com/en-us/windows/win32/api/processtopologyapi/nf-processtopologyapi-getprocessgroupaffinity) function retrieves the set of groups to which a process and its threads are assigned.

[Here's](https://learn.microsoft.com/en-us/windows/win32/procthread/processor-groups#:~:text=Affinity%20APIs%20that%20are%20not%20group%2Daware%20or%20operate%20on%20a%20single%20group%20implicitly%20use%20the%20primary%20group%20as%20the%20process/thread%20processor%20group%3B%20for%20more%20information%20on%20the%20new%20behaviors%20check%20the%20Remarks%20sections%20for%20the%20following%3A) the link for Affinity APIs (also included in reference [1]).

Pros: 

- Possible to set affinities on creation.
- Child processes inherit parents' affinities.

Cons:

- Hard to set multi-group process affinities (you need to manage each individual thread).
- Legacy API with less intuitive interface than CPU Sets APIs.


## References 


### Processor groups, Affinity APIs, and CPU Sets APIs:

[1] [Processor Groups - Microsoft Learn](https://learn.microsoft.com/en-us/windows/win32/procthread/processor-groups) (for functions that belong to Affinity APIs, Ctrl+F for "Affinity APIs that are not group-aware or operate")

[2] [CPU Sets - Microsoft Learn](https://learn.microsoft.com/en-us/windows/win32/procthread/cpu-sets)

### Process creation:

[3] [visual c++ - What is the difference between CreateProcess and CreateProcessA? - Stack Overflow](https://stackoverflow.com/questions/3060991/what-is-the-difference-between-createprocess-and-createprocessa)

[4] [Genesis - The Birth of a Windows Process (Part 1)](https://fourcore.io/blogs/how-a-windows-process-is-created-part-1)

[5] [Genesis - The Birth of a Windows Process (Part 2)](https://fourcore.io/blogs/how-a-windows-process-is-created-part-2)

[6] [Create Processes - Microsoft Learn](https://learn.microsoft.com/en-us/windows/win32/procthread/creating-processes)

### PowerShell method: 

[7] [batch file - Change affinity of process with windows script - Stack Overflow](https://stackoverflow.com/questions/19187241/change-affinity-of-process-with-windows-script)

[8] [Process Class (System.Diagnostics) - Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.process?view=net-8.0)

### `start` method:

[9] [shortcuts - How do I set the group and affinity of a Windows executable from the command line - Super User](https://superuser.com/questions/1156840/how-do-i-set-the-group-and-affinity-of-a-windows-executable-from-the-command-lin)

[10] [start - Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/start)
