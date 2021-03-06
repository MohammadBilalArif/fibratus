Fibratus
========

![fibratus logo]( https://github.com/rabbitstack/fibratus/blob/master/fibratus.png "fibratus logo" )

[![Build status](https://ci.appveyor.com/api/projects/status/dlvxhc0j026ikcyv?svg=true)](https://ci.appveyor.com/project/rabbitstack/fibratus)
[![Coverage Status](https://codecov.io/gh/rabbitstack/fibratus/branch/master/graph/badge.svg)](https://codecov.io/gh/rabbitstack/fibratus)
[![Code Health](https://landscape.io/github/rabbitstack/fibratus/master/landscape.svg?style=flat)](https://landscape.io/github/rabbitstack/fibratus/master)

**Fibratus** is a tool which is able to capture the most of the Windows kernel activity - process/thread creation and termination, 
file system I/O, registry, network activity, DLL loading/unloading and much more. 
Fibratus has a very simple CLI which encapsulates the machinery to start the kernel event stream collector, 
set kernel event filters or run the lightweight Python modules called **filaments**. You can use filaments to extend Fibratus with your own arsenal of tools.

![fibratus]( https://github.com/rabbitstack/fibratus/blob/master/static/fibratus.png "fibratus" )
![filaments]( https://github.com/rabbitstack/fibratus/blob/master/static/filaments.png "filaments" )


## Installation

Install via the pip package manager:

`pip install fibratus`

## Running Fibratus

Run `fibratus --help` for usage instructions:

```
Usage:
    fibratus run ([--filament=<filament>] | [--filters <kevents>...]) [--no-enum-handles]
    fibratus list-kevents
    fibratus list-filaments
    fibratus -h | --help
    fibratus --version

Options:
    -h --help                 Show this screen.
    --filament=<filament>     Specify the filament to execute.
    --no-enum-handles         Avoids enumerating the system handles.
    --version                 Show version.
```
To capture all of the supported kernel events, execute `fibratus run` command without any argument. After the collector has been initialized, the continuous stream of kernel events will render on the standard output.

```
5550 20:28:14.882000 3 cmd.exe (4396) - UnloadImage (base=0x77950000, checksum=1313154, image=ntdll.dll, path=\Device\HarddiskVolume2\Windows\SysWOW64\ntdll.dll, pid=4396, size=1536.0)
5551 20:28:14.882000 3 erl.exe (2756) - TerminateProcess (comm=C:\Windows\system32\cmd.exe /cdir /-C /W c:/Users/Nedo/AppData/Roaming/RabbitMQ/db/rabbit@NEDOPC-mnesia, exe=C:\Windows\system32\cmd.exe, name=cmd.exe, pid=4396, ppid=2756)
5552 20:28:14.882000 3 erl.exe (2756) - CloseFile (file=\Device\HarddiskVolume2\Windows, tid=1672)
5631 20:28:17.286000 2 taskmgr.exe (3532) - RegQueryKey (hive=REGISTRY\MACHINE\SYSTEM, key=ControlSet001\Control\Nls\Locale, pid=3532, status=0, tid=4324)
5632 20:28:17.286000 2 taskmgr.exe (3532) - RegOpenKey (hive=REGISTRY\MACHINE\SYSTEM, key=ControlSet001\Control\Nls\Locale\Software\Microsoft\DirectUI, pid=3532, status=3221225524, tid=4324)
5633 20:28:17.288000 2 taskmgr.exe (3532) - CreateFile (file=\Device\HarddiskVolume2\Windows\system32\xmllite.dll, file_type=REPARSE_POINT, operation=OPEN, share_mask=rwd, tid=4324)
5634 20:28:17.288000 2 taskmgr.exe (3532) - CloseFile (file=\Device\HarddiskVolume2\Windows\system32\xmllite.dll, tid=4324)
5635 20:28:17.288000 2 taskmgr.exe (3532) - CreateFile (file=\Device\HarddiskVolume2\Windows\system32\xmllite.dll, file_type=FILE, operation=OPEN, share_mask=r-d, tid=4324)
5636 20:28:17.288000 2 taskmgr.exe (3531) - LoadImage (base=0x7fefab90000, checksum=204498, image=xmllite.dll, path=\Windows\System32\xmllite.dll, pid=3532, size=217088)
5637 20:28:17.288000 2 taskmgr.exe (3532) - CloseFile (file=\Device\HarddiskVolume2\Windows\system32\xmllite.dll, tid=4324)
5638 20:28:17.300000 2 taskmgr.exe (3532) - RegQueryKey (hive=REGISTRY\MACHINE\SYSTEM, key=ControlSet001\Control\Nls\Locale\, pid=3532, status=0, tid=4324)
5639 20:28:17.300000 2 taskmgr.exe (3532) - RegOpenKey (hive=REGISTRY\MACHINE\SYSTEM, key=ControlSet001\Control\Nls\Locale\SOFTWARE\Microsoft\CTF\KnownClasses, pid=3532, status=3221225524, tid=4324)
5640 20:28:17.300000 3 taskmgr.exe (3532) - RegQueryKey (hive=REGISTRY\MACHINE\SYSTEM, key=ControlSet001\Control\Nls\Locale\, pid=3532, status=0, tid=4324)
5641 20:28:17.300000 3 taskmgr.exe (3532) - RegOpenKey (hive=REGISTRY\MACHINE\SYSTEM, key=ControlSet001\Control\Nls\Locale\SOFTWARE\Microsoft\CTF\KnownClasses, pid=3532, status=3221225524, tid=4324)
5642 20:28:17.302000 2 taskmgr.exe (3532) - UnloadImage (base=0x7fefab90000, checksum=204498, image=xmllite.dll, path=\Windows\System32\xmllite.dll, pid=3532, size=212.0)
````
Hit <kbd>Ctrl</kbd>+<kbd>C</kbd> to stop Fibratus. Note that depending on the system load, you might have to hit <kbd>Ctrl</kbd>+<kbd>C</kbd> **multiple** times until kernel event buffers are consumed.

Every line contains the information of the kernel event according to the following format:

* `id` - kernel event's incremental identifier. The value of the identifier is reseted on every single execution.
* `timestamp` - temporal occurrence of the event.
* `cpu` - the CPU core where the event has been generated.
* `process` - process name which triggered the kernel's event.
* `pid` - the identifier of the after-mentioned process.
* `kevent` - name of the kernel event.
* `params` - event's parameters.

### Filtering

Fibratus supports basic filtering capabilities on kernel event names. To capture the specified kernel events, use `fibratus run --filters` command. For example, `fibratus run --filters CreateProcess Send` would capture the events related to process creation and data sending over network sockets. For a full list of kernel events see the table below.

| Kernel event        | Description                     | 
| ------------------- |:------------------------------- | 
| CreateProcess       | Creates a new process and its primary thread  |
| CreateThread        | Creates a thread to execute within the virtual address space of the calling process |
| TerminateProcess    | Terminates the process and all of its threads   |
| TerminateThread     | Terminates a thread  |
| LoadImage           | Loads the module into the address space of the calling process |
| UnloadImage         | Frees the loaded module from the address space of the calling process |
| CreateFile          | Creates or opens a file or I/O device |
| CloseFile           | Closes the file or I/O device |
| DeleteFile          | Deletes an existing file or directory |
| RenameFile          | Renames a file or directory |
| ReadFile            | Reads data from the file or I/O device   |
| WriteFile           | Writes data to the file or I/O device  |
| Send                | Sends data on a connected socket |
| Recv                | Receives data from a connected socket  |
| Accept              | Initiates the connection attempt from the remote or local TCP socket |
| Connect             | Establishes the connection to a TCP socket  |
| Disconnect          | Closes the connection to a TCP socket |
| Reconnect           | Reconnects to a TCP socket |
| RegCreateKey        | Creates the registry key or open it if the key already exists  |
| RegQueryKey         | Retrieves information about the registry key ||
| RegOpenKey          | Opens the registry key |
| RegDeleteKey        | Deletes a subkey and its values |
| RegQueryValue       | Retrieves the type and data of the value associated with an open registry key |
| RegSetValue         | Sets the data and type of a value under a registry key |
| RegDeleteValue      | Removes a value from the registry key |

### Executing filaments

Filaments are micro modules written in Python that run on top of Fibratus in its own execution context. They often perform aggregations, filtering, groupings, counting or any kind of custom logic on a kernel event stream. To execute a filament, pass the filament name via `--filament` argument, for example `fibratus run --filament=top_hives_io`. To get more information on how to create filaments, see [Building filaments](#building-filaments). 

## Building filaments

Creating a new filament is as easy as providing a Python module with `on_init` and `on_next_kevent` methods as follows:

```python
import collections

"""
Shows the top TCP / UDP outbound packets.
"""

connections = collections.Counter()


def on_init():
    set_filter('Send')
    columns(['Destination', 'Count'])
    sort_by('Count')
    set_interval(1)
    title('Top outbound TCP/UDP packets')


def on_next_kevent(kevent):
    dst = ['%s:%d' % (kevent.params.ip_dst, kevent.params.dport)]
    connections.update(dst)


def on_interval():
    for ip, count in connections.items():
        add_row([ip, count])
    render_tabular()
```
The `on_init` method is invoked upon Fibratus initialization just before the kernel event stream is being opened. When a new kernel event arrives, it is delivered to the filament via `on_next_kevent` function. To report the information on a periodic basis the filament module provides the `on_interval` callback.

### Streaming kernel events

Fibratus has a set of built-in adapters for emitting the kernel events to external endpoints. You can choose between these transports:

- AMQP 
```python

def on_next_kevent(kevent):
    body = {'ip_dst': kevent.params.ip_dst, 'dport': kevent.params.dport}
    amqp.emit(body)
    # overrides the default exchange and the routing key
    amqp.emit(body, exchange='amqp.direct', routingkey='net')
```

- SMTP
```python

import json

def on_next_kevent(kevent):
    body = json.dumps({'ip_dst': kevent.params.ip_dst, 'dport': kevent.params.dport})
    smtp.emit(body, subject='network packet received from %s' % kevent.params.ip_src)
```

For more details see the `fibratus.yml` configuration file.

## Contributing

Please use Github's pull-request model to submit your contributions. Before you send the pull-request you should keep in mind:

* the code has to be in harmony with the **Zen Of Python** principles
* you need to test the code (fibratus uses the `pytest` unit testing framework)
* make sure your code satisfy the **PEP** coding style

## License

Copyright 2015/2016 by Nedim Sabic (RabbitStack) 
All Rights Reserved. 

http://rabbitstack.github.io

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.

## Support on Beerpay
Hey dude! Help me out for a couple of :beers:!

[![Beerpay](https://beerpay.io/rabbitstack/fibratus/badge.svg?style=beer-square)](https://beerpay.io/rabbitstack/fibratus)  [![Beerpay](https://beerpay.io/rabbitstack/fibratus/make-wish.svg?style=flat-square)](https://beerpay.io/rabbitstack/fibratus?focus=wish)