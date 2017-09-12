# How to Analyse Linux/arm32 coredump on Linux/x64 Host

A chroot can be used to analyse Linux/arm32 coredump on Linux/x64 host.

## Prerequisites

Tool is based on "build" package (gbs uses this package too) so to use it it's necessary to install following dependencies on Linux x64 host:

- rpm
- libxml-parser-perl
- libcrypt-ssleay-perl
- binfmt-support
- qemu-arm-static

```
$ sudo apt-get install rpm libxml-parser-perl libcrypt-ssleay-perl binfmt-support qemu-arm-static
```

## Building chroot

```bash
./build_chroot.py -B /media/kbaladurin/data/CHROOT \
                  -R http://download.tizen.org/snapshots/tizen/base/latest/repos/arm/packages/ \
                     http://download.tizen.org/snapshots/tizen/unified/latest/repos/standard/packages/ \
                  --clean
```

### Options

| &nbsp;&nbsp;&nbsp;&nbsp;Name&nbsp;&nbsp;&nbsp;&nbsp; | Description |
|:---:| --- |
| -B      | This option is used to specify chroot location |
| -R      | This option is used to specify *.rpm repositories. It's necessary to specify at least following repos: [http://download.tizen.org/snapshots/tizen/base/<version\>/repos/arm/packages/]() [http://download.tizen.org/snapshots/tizen/unified/<version\>/repos/standard/packages/]() | 
| --clean | This option is used to rebuild chroot |

The chroot contains `lldb` and `coreclr` packages by default. Other necessary file can be copied to it.

## Debugging coredump

You can enter the arm32 shell environment with the following command.

```bash
$ sudo chroot /media/kbaladurin/data/CHROOT/local/BUILD-ROOTS/scratch.armv7l.0
bash-3.2# 
```

Now, you can debug coredump using lldb as in the arm32 target.

```
bash-3.2# /home/owner/share/tmp/sdk_tools/lldb/bin/lldb -c core.9326
(lldb) target create --core "core.9326"
Core file
'/usr/share/dotnet/shared/Microsoft.NETCore.App/2.0.0/core.9326' (arm) was loaded.
(lldb) bt
* thread #1: tid = 0, 0xb6bfe094 libc.so.6`gsignal + 156, name = 'corerun', stop reason = signal SIGABRT
   * frame #0: 0xb6bfe094 libc.so.6`gsignal + 156
     frame #1: 0xb6bff3f0 libc.so.6`abort + 300
     frame #2: 0xb69fc4e4 libcoreclr.so`::PROCAbort() + 16 at process.cpp:3046
     frame #3: 0xb69fb7de libcoreclr.so`PROCEndProcess(hProcess=<unavailable>, uExitCode={UINT}, bTerminateUnconditionally=YES) + 222 at process.cpp:1394
     frame #4: 0xb6823e24 libcoreclr.so`UnwindManagedExceptionPass1(ex=<unavailable>, frameContext=<unavailable>) + 460 at exceptionhandling.cpp:4681
     frame #5: 0xb6823f0e libcoreclr.so`DispatchManagedException(ex=<unavailable>, isHardwareException=<unavailable>) + 158 at exceptionhandling.cpp:4752
     frame #6: 0xb67dd1ca libcoreclr.so`IL_Throw(obj=<unavailable>) + 466 at jithelpers.cpp:5309
     frame #7: 0xafac567c JIT(0x2022a88)`MyStaticCallbackWithException_1 + 104
     frame #8: 0xb687b440 libcoreclr.so`CallEHFunclet + 24
     frame #9: 0xb68205a4 libcoreclr.so`ExceptionTracker::CallCatchHandler(_CONTEXT*, bool*) [inlined] ExceptionTracker::CallHandler(this=<unavailable>, uHandlerStartPC={UINT_PTR}, pEHClause=<unavailable>, pMD=<unavailable>, funcletType=<unavailable>) + 82 at exceptionhandling.cpp:3401
     frame #10: 0xb6820552 libcoreclr.so`ExceptionTracker::CallCatchHandler(this={ExceptionTracker}, pContextRecord=<unavailable>, pfAborting=<unavailable>) + 86 at exceptionhandling.cpp:646
     frame #11: 0xb6820daa libcoreclr.so`::ProcessCLRException(pExceptionRecord={_EXCEPTION_RECORD}, MemoryStackFp=<unavailable>, pContextRecord=<unavailable>, pDispatcherContext=<unavailable>) + 1030 at exceptionhandling.cpp:1177
     frame #12: 0xb6823ac0 libcoreclr.so`UnwindManagedExceptionPass2(ex=<unavailable>, unwindStartContext=<unavailable>) + 256 at exceptionhandling.cpp:4483
     frame #13: 0xb6823e52 libcoreclr.so`UnwindManagedExceptionPass1(ex=<unavailable>, frameContext=<unavailable>) + 506 at exceptionhandling.cpp:4698
     frame #14: 0xb6823f0e libcoreclr.so`DispatchManagedException(ex=<unavailable>, isHardwareException=<unavailable>) + 158 at exceptionhandling.cpp:4752
     frame #15: 0xb681fffe libcoreclr.so`HandleHardwareException(ex={PAL_SEHException}) + 434 at exceptionhandling.cpp:5275
     frame #16: 0xb69d5776 libcoreclr.so`SEHProcessException(exception={PAL_SEHException}) + 98 at seh.cpp:283
     frame #17: 0xb69d67a4 libcoreclr.so`common_signal_handler(code=<unavailable>, siginfo=<unavailable>, sigcontext=<unavailable>, numParams=<unavailable>) + 200 at signal.cpp:880
     frame #18: 0xb69d66a6 libcoreclr.so`::signal_handler_worker(code=<unavailable>, siginfo={siginfo_t}, context=<unavailable>, returnPoint={SignalHandlerWorkerReturnPoint}) + 78 at signal.cpp:428
     frame #19: 0xb69ff178 libcoreclr.so`CallSignalHandlerWrapper0 + 4
     frame #20: 0xafac5580 JIT(0x2022a88)`MyStaticCallbackWithException + 128
     frame #21: 0xafac54ea JIT(0x2022908)`IL_STUB_ReversePInvoke + 26
     frame #22: 0xb687aacc libcoreclr.so`UMThunkStub + 98
     frame #23: 0xafab17a0
(lldb) clrthreads
ThreadCount:      2
UnstartedThread:  0
BackgroundThread: 1
PendingThread:    0
DeadThread:       0
Hosted Runtime:   no
Lock
        ID OSID ThreadOBJ    State GC Mode     GC Alloc Context  
Domain   Count Apt Exception
XXXX    1 2470 01FC1A88     21220 Preemptive 00000000:00000000 01FBCC48 
0     Ukn (Finalizer)
XXXX    2 246e 01FD31C0     20020 Preemptive FFFFFFFF:B232337C 01FBCC48 
0     Ukn <Invalid Object> (b230aca8) (nested exceptions)
```
