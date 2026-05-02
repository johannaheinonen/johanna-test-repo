# Linux Shell Scripting Basics
## What is a Linux shell?
Linux shell is a program that provides an interface for the user to use operating system services. It interpretes the commands from users and passes them to Linux kernel to execute. 
- shell - the command interpreter (e.g. bash)
- terminal - application where you type the commands  


### Bash shell
Bash is a default shell on most Linux distributions and __~/.bashrc__ is its startup configuration file. It runs every time you open a new terminal window in Bash. In this file it is possible to specify for example   
- aliases (shortcuts for commands, for example ```alias ll='ls -l'```  or ```alias gs='git status'```)  
- Functions  
- Prompt settings (how your terminal prompt looks)
  
Notice:  
- ```.bashrc``` runs every time you open a terminal 
- ```.profile``` runs once when you log in (e.g. ssh session or login to GUI)  



## What is a shell script?
A shell script is a text file containing shell commands. When executed, the shell reads the file and runs each command.

Shell scripts 
  - are used to automate repetitive tasks which saves time and reduces errors
  - make it easy to link different commands together.

Example:  
```nano helloworld.sh```  

```#!/bin/bash      # this tells the system which shell to use to interpret the script```  
```echo "Hello, world!"```  

How to run a shell script?
1.  make the script executable:
   
```ls -l helloworld.sh```  
```-rw-rw-r-- 1 testuser testuser   34 20. 9. 16:07 helloworld.sh```  

```chmod ugo+x helloworld.sh```  
```ls -l helloworld.sh```    
```-rwxrwxr-x 1 testuser testuser   34 20. 9. 16:07 helloworld.sh```  

__notice__:  
 - x permission - it is possible to run the script.
 - r permission - it is possible to view the contents of the script.
 - w permission - it is possible to modify the script.  

2. run the script:

```pwd```  
```/home/testuser```  

```helloworld.sh```  
```bash: helloworld.sh: command not found```  
Bash tries to find an executable named ```helloworld.sh``` in the directories listed in environment variable ```$PATH```.  
```echo $PATH```  
```/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games```  

Because user's home directory is not in that list the location of the script has to be told explicitly:  

```./helloworld.sh```  
```Hello, world!```  

- ```.``` means 'this directory'
- ```/``` is the path separator
- ```./helloworld.sh``` means run the script in this directory 

__Notice__: ```bash helloworld.sh``` works because in this case you are starting a new bash process (a subshell) that is in the ```$PATH``` (```/bin```), this subshell executes the script that located in the current working directory and terminates a subshell when the script finishes.  


__Notice2__: environment variable ```$PATH``` is usually __not__ changed. As an exception user specific ```$HOME/bin/``` may be created and added to the ```$PATH```:  
   ```export PATH="$PATH:$HOME/bin"```  
   To make this permanent reload the ```.profile``` file (make sure that directory ```$HOME/bin``` exists).      
   ```source ~/.profile```   
   
__Notice3__: If you create a script that everybody should be able to run copy it to ```/usr/local/bin/```  

## Variables and Input  

A variable in shell scripting is a name that stores a value. To create a variable, use the following syntax: 
```VARIABLE_NAME=VALUE``` (capital letters are commonly used in environment variables like ```$HOME``` or ```$PATH``` or global settings)  

Example: ```username=testuser``` creates a variable ```username``` and assigns value ```testuser``` to it.  
        (lowercase letters are commonly used in script internal variables)  

Notice:  
 - variable names are case sensitive, ```name``` and ```Name``` are two different variables.  
 - if the value is more than one word use quotes:  ```greeting='How are you?'```  
 - To refer a variable __value__ use the dollar sign ($), for example:  
       ```username=testuser```  
       ```echo $username```  
       ```testuser```  
 - without $ the outcome is the name of the variable.  
       ```echo username```  
       ```username```  


Example: ```listvariables.sh```  
   ```
   #!/bin/bash
   var1=1
   var2=2
   var3=3
   echo The first three numbers are $var1 $var2 and $var3

   ./listvariables.sh
   The first three numbers are 1 2 and 3
   ```  

It is also possible to pass variables as command line arguments: ```list3numbers.sh```   
   ```
   #!/bin/bash
   var1=$1
   var2=$2
   var3=$3
   echo Three numbers are $var1 $var2 $var3

   ./list3numbers.sh 56 37 3
   Three numbers are 56 37 3

   ./list3numbers.sh
   Three numbers are 
   ```  

Asking input from user is also an option: ```hello.sh```  
```
   #!/bin/bash  
   read -p "Who are you? " name  
   echo "Hello, $name"  

   ./hello.sh
   Who are you? testuser
   Hello, testuser

   ```  

## Conditional Statements: if-then-else

```
if [condition]; then   
  commands if condition is true    
else   
  commands if condition is false  
fi  
```  

Condition can be for example:  
- -eq - Equal
- -gt - Greater than
- -lt - Less than
- -n - not empty  


Example 1: Package Update Notifier  
```
#!/bin/bash

# Update package list - very quiet mode
sudo apt-get update -qq

# Check for upgradable packages, do not install
updates=$(sudo apt-get --just-print upgrade | grep "^Inst")

# check whether variable updates is empty (-n means 'not empty')
if [ -n "$updates" ]; then
  echo "Updates available:"
  echo "$updates"
else
  echo "System is up to date."
fi
```
```
./updateNotifier.sh 
Updates available:  
Inst base-files [13.8] (13.8+deb13u1 Debian:13.1/stable [amd64])  
Inst init-system-helpers [1.68] (1.69~deb13u1 Debian:13.1/stable [all])  
Inst init [1.68] (1.69~deb13u1 Debian:13.1/stable [amd64])  
. . .  

./updateNotifier.sh  
System is up to date.  

```  
Example 2: ```.profile file```  

## Loops 
For loop:  
```  
for [condition]; do    
    commands  
done  
```  
While loop:  
```  
while [condition]; do  
    commands  
done  
```  

Example1: Print numbers 1 to 10  
```  
#!/bin/bash  

for i in {1..10}; do  
  echo "Number: $i"  
done  
```
```  
./printnumbers.sh 
Number: 1  
Number: 2  
Number: 3  
Number: 4  
Number: 5  
Number: 6  
Number: 7  
Number: 8  
Number: 9  
Number: 10
```  

Example2 - infinitive loop:  
```
#!/bin/bash  

while true; do  
  echo "Current time: $(date)"  
  sleep 5  
done  
```
  
```  
./infinitiveloop.sh   
Current time: to 25.9.2025 12.17.10 +0300  
Current time: to 25.9.2025 12.17.15 +0300  
Current time: to 25.9.2025 12.17.20 +0300  
. . .
```

Example 3: Monitor free disk space  
```
#!/bin/bash  
while true; do  
    df -h /dev/sda1  
    echo "-----------------------------"  
    sleep 5  
done

/diskusage.sh 
Filesystem      Size  Used Avail Use% Mounted on  
/dev/sda1        16G  9,9G  4,8G  68% /  
-----------------------------  
Filesystem      Size  Used Avail Use% Mounted on  
/dev/sda1        16G  9,9G  4,8G  68% /  
-----------------------------  

```
To test this script create a dummy file (size 100M) in another terminal: ```dd if=/dev/zero of=dummy_file.bin bs=1M count=100```  

# Linux Automation  
Automating routine tasks is essential when trying to increase the efficiency, reliability, consistency and scalability of the system. Linux automation can vary from simple shell sripting to complex configurations. Common Linux automation tools:    
 - __shell scripts__ - can be used for example to backup important files on a regular basis. Script can perform this task regularly and automatically if scheduled to run as a cron job.  
 - __configuration management tools__ like ansible can be used to create the initial server setup (users, groups, ssh keys, sudo access etc.), install the required software packages, start required services, security hardening (fw rules, disable root login etc.), install monitoring agents, configure log rotation and many other tasks.  
 - __provisioning tools__ like terraform (Infrastructure as a code, IaC) can be used for example in cloud deployments to provision the virtual machines.  


