# Create a Trojan

## Backdooring a code cave with Putty POC

There are 3 areas a PE can be backdoored:

·         Code cave

·         Create new section

·         Extend a section

We’ll be focused on creating a backdoor in the code cave for this example.

Download Putty and x32Dbg [https://x64dbg.com/](https://x64dbg.com/#)

### Find Code Cave

Open Putty with debugger

You’ll notice the first breakpoint is set before the process is initialized.

![](<../.gitbook/assets/image (181).png>)

![](<../.gitbook/assets/image (182).png>)

We’ll focus on this portion of the code so the backdoor can be executed before putty is initialized.

Our code cave will reside in the .text section

![](<../.gitbook/assets/image (173).png>)

Code Cave Start Address 0045C961

![](<../.gitbook/assets/image (169).png>)

Set a break point at this address so it can be referenced for later in the breakpoints tab.

![](<../.gitbook/assets/image (170).png>)

### Adding shellcode

Head on back to the entry point and add a jmp call to your code cave where the shellcode will reside. Before adding the jmp call, copy the first few calls since they are about to be overwritten and you’ll need to maintain the functionality of putty.

![](<../.gitbook/assets/image (183).png>)

&#x20;Add jmp call where the entry point is.

![](<../.gitbook/assets/image (180).png>)

![](<../.gitbook/assets/image (178).png>)

&#x20;Now at the start of your code cave you will want to change the instructions to save the registers and flags

![](<../.gitbook/assets/image (172).png>)

Now you can add your shellcode by highlighting a section large enough to fit your shellcode and then add it by Right Clicking the highlighted section and selecting Binary -> Edit.

![](<../.gitbook/assets/image (185).png>)

![](<../.gitbook/assets/image (174).png>)

You now have shellcode inserted into putty and if this was saved and ran, the shellcode would run but putty would not be initialized so we will have to identify which instruction in the shellcode terminates the process and overwrite that with a jmp call back to the start of putty. You can confirm that your shellcode is running by either saving the patches made and running the new PE or running through each breakpoint. At the end you should notice the shellcode run (calc opens in this case) and the debugger is blank (the process terminated), if you ran through each breakpoint.

![](<../.gitbook/assets/image (176).png>)

### Jmp back to Putty

To find the instruction that exits the process, set a breakpoint on every call instruction in the shellcode. You will then need to step through each breakpoint to see which one terminates the process.

![](<../.gitbook/assets/image (177).png>)

In this case, the call instruction at 0045C9FA opens calc and the call instruction at 0045CA19 terminates the process. &#x20;

![](<../.gitbook/assets/image (171).png>)

&#x20;The final step will be to skip the final call instruction and jump back to the original putty. To do this, replace the push 0 instruction before the final call instruction with a jmp instruction to an address in the code cave with room to modify it’s contents. In this section we will add the registers and flags that were saved and some of the code that was overwritten by the first jump instruction.

![](<../.gitbook/assets/image (179).png>)

![](<../.gitbook/assets/image (184).png>)

In the screenshot above, you’ll see that the registers and flags were added back, and then the instructions that were overwritten were added back before jumping back to the original putty code. Now patch the file and run it. You’ll see the putty and calc.exe.

&#x20;
