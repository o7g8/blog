+++
title = "Remote debugging of C/C++ code with Visual Studio Code"
date = 2018-09-23T23:00:45Z
draft = false
tags = ["Visual Studio Code", "Linux", "debugging", "C", "C++", "gcc", "gdb"]
categories = []
+++

I'm currently porting a proprietary C/C++ financial calculations library to Linux. During the work I need to debug the code on a remote Linux machine.

In the article I will describe two ways of remote debugging of C/C++ code executed on Linux using Visual Studio Code.

All the examples below assume Ubuntu 16.04.

## Way 1: Debugging with VSCode/Windows via SSH to a Linux host

With this approach your source code and Visual Studio Code reside on Windows machine, and the program you debug resides on a Linux machine. 

On the Windows host you will need:

* Visual Studio Code with the following extensions: C/C++, Native Debug;

* Installed [MSYS2](https://www.msys2.org/) with MINGW `gcc` compiler and `gdb` debugger;

* MINGW should be added into your Windows `PATH`.

On the Linux machine you will need:

* The binary to debug;

* Installed `gdb` debugger.

Add a new remote debugging session in your VSCode/Windows `launch.json` using the example below (the example uses Linux machine `vagrant` with user `vagrant` and password `vagrant`):

```json

{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        
        {
            "name": "SSH Remote Debug",
            "type": "gdb",
            "request": "launch",
            "target": "<path-to-binary-to-debug-on-linux-machine>",
            "arguments": "<binary-argumets>",
            "cwd": "${workspaceRoot}",
            "ssh": {
                "forwardX11": true,
                "host": "vagrant",
                "cwd": "<working-directory-on-linux>",
                //"keyfile": "/path/to/.ssh/key", // OR
                "password": "vagrant",
                "user": "vagrant",
                "x11host": "localhost",
                "x11port": 6000,
                // Optional, content will be executed on the SSH host before the debugger call.
                //"bootstrap": "source /home/remoteUser/some-env"
            }
        }
    ]
}

```

Set some breakpoints in your VSCode/Windows and start the debugging session.

## Way 2: VSCode/Linux with X11 forwarding to a Windows machine

With this approach your source code, Visual Studio Code and the binary you need to debug all reside on a remote headless Linux machine. 

On the Windows machine you will need only a X Server.

* Install [Xming X Server](https://sourceforge.net/projects/xming/) on your Windows machine and start it.

* Make VS Code repository available in your Linux machine as it's described in the [Visual Studio Code on Linux](https://code.visualstudio.com/docs/setup/linux).

* Install the VS Code along with its dependencies and software necessary for the debugging on the headless Linux machine:

```bash
sudo apt install code libxss1 libasound2 xterm gdb
```

The `xterm` is necessary for debugging and we also will use it to ensure the X11 forwarding and the X Server are working.

* Start a `Putty` session on the Linux machine, ensure the checkbox `Connection > SSH > X11 > Enable X11 forwarding` is checked.

* First ensure your X11 forwarding and X Server work and start `xterm` on the Linux machine:

```bash

xterm

```

You should get a new terminal window on your Windows desktop.

* Now start the VS Code `code` on Linux. If nothing happens start it again in verbose mode:

```bash

code --verbose

```

If you get an error (typical for Ubuntu 16.04):

```bash

...
code: ../../src/xcb_io.c:274: poll_for_event: Assertion `!xcb_xlib_threads_sequence_lost' failed.

```

Then do the following workaround:

```bash

cd /usr/share/code
sudo cp /usr/lib/x86_64-linux-gnu/libxcb.so.1 .
sudo sed -i 's/BIG-REQUESTS/_IG-REQUESTS/' libxcb.so.1

```

* Start the `code` again. You should get the VS Code windows on your Windows desktop. Install C/C++ extension.

Now you can debug your Linux code. For the VS Code it will be a regular local debugging session.

Configure the debugging session in VS Code `launch.json` as:

```json

{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "(gdb) Launch",
            "type": "cppdbg",
            "request": "launch",
            "program": "<path-to-binary-to-debug-on-linux-machine>",
            "args": ["<binary-argumets>"],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": true,
            "MIMode": "gdb",
            "miDebuggerPath": "/usr/bin/gdb",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ]
        }
    ]
}

```
