# container-security-notes
Some short notes I made while reading Container Security by Liz Rice 

## Linux system calls, permissions and capabilities

- applications run in user space, and to execute a elevated operation like reading from a file, making a network request, getting time, these programs need to communicate with kernel which has whole priviledged access to memory and processing
- this communication interface is `syscall` interface `~400` calls exist *
- Example - `read`, `write`, `open`, `execve`, `chown`, `clone`, `fork` etc.
- `FILE PERMISSIONS` IN LINUX WORK AS FOLLOWED

- ![file permissions](./assets/file-permissions.png) 
- `SETUID`
  - a special permission bit in place of `x` bit in permission list that allows the executable to be ran as owner of the file, example **`sudo`** is owned by **`root`** user therefore we use it run any other command as a root user rather than login in as root
  
  	```bash
	╭─prathamesh at SOLITUDE in ~/github 26-02-08 - 19:26:31
	╰─○ ls -lh $(which sudo)
	Permissions Size User Date Modified Name
	.rwsr-xr-x  232k root 25 Jun  2025  /usr/bin/sudo*
  	```
  - this section then contains example of running our own copy of **`sleep`** command where we make root as owner of new **`mysleep`** and giving set it's setuid and executing it as normal user
	<details>
	<summary><b>setuid example with sleep</b></summary>

	```bash
	╭─prathamesh at SOLITUDE in ~ 26-02-08 - 19:34:29
	╰─○ ls -lh $(which sleep)
	Permissions Size User Date Modified Name
	.rwxr-xr-x   35k root  8 Feb  2024  /usr/bin/sleep

	╭─prathamesh at SOLITUDE in ~ 26-02-08 - 19:34:46
	╰─○ cp $(which sleep) ./mysleep

	╭─prathamesh at SOLITUDE in ~ 26-02-08 - 19:35:03
	╰─○ ls -lh mysleep
	Permissions Size User       Date Modified Name
	.rwxr-xr-x   35k prathamesh  8 Feb 19:35  mysleep

	╭─prathamesh at SOLITUDE in ~ 26-02-08 - 19:35:14
	╰─○ sudo chown root ./mysleep

	╭─prathamesh at SOLITUDE in ~ 26-02-08 - 19:35:23
	╰─○ ls -lh mysleep
	Permissions Size User Date Modified Name
	.rwxr-xr-x   35k root  8 Feb 19:35  mysleep

	╭─prathamesh at SOLITUDE in ~ 26-02-08 - 19:35:38
	╰─○ sudo chmod +s ./mysleep

	╭─prathamesh at SOLITUDE in ~ 26-02-08 - 19:35:43
	╰─○ ls -lh mysleep
	Permissions Size User Date Modified Name
	.rwsr-sr-x   35k root  8 Feb 19:35  mysleep

	╭─prathamesh at SOLITUDE in ~ 26-02-08 - 19:35:45
	╰─○ sleep 10 &
	[1] 4766

	╭─prathamesh at SOLITUDE in ~ 26-02-08 - 19:36:00
	╰─○ ps -fC sleep
	UID          PID    PPID  C STIME TTY          TIME CMD
	pratham+    4766     703  0 19:36 pts/0    00:00:00 sleep 10

	╭─prathamesh at SOLITUDE in ~ 26-02-08 - 19:36:05
	╰─○ ./mysleep 10 &
	[1]  + 4766 done       sleep 10

	╭─prathamesh at SOLITUDE in ~ 26-02-08 - 19:36:10
	╰─○ ./mysleep 10 &
	[1] 4836

	╭─prathamesh at SOLITUDE in ~ 26-02-08 - 19:36:12
	╰─○ ps -fC mysleep
	UID          PID    PPID  C STIME TTY          TIME CMD
	root        4836     703  0 19:36 pts/0    00:00:00 ./mysleep 10
	```
	</details>
