---
draft: false
date:
  created: 2025-06-04
  updated: 2025-06-06
categories:
  - clipboard
  - linux
  - gtk
  - desktop
  - wayland
  - x11
---


# :material-clipboard-flow: Clipboard Snooping (X11 and Wayland)

This post takes a look at what's possible in terms of "clipboard snooping" or stealing and setting content to the clipboard in both, X11 and Wayland display servers on Linux.

<!-- more -->

!!! warning "Under Construction"

    This page is still a work-in-progress, and will be updated over time.

!!! warning "AI Usage"

    AI was used to create and share code snippets, leading to the documentation and other public examples referenced in this section.

First determine which display server you're using: `echo $XDG_SESSION_TYPE`


## PyGObject Libraries

The PyGObject bindings, if installed, allow you to interact with the GNOME / GTK clipboard contents.

- [GTK4 Documentation: Getting Started](https://pygobject.gnome.org/getting_started.html)
- [GTK4 Documentation: Clipboard Example](https://pygobject.gnome.org/tutorials/gtk4/clipboard.html)
- [GTK3 Documentation: Clipboard Example](https://python-gtk-3-tutorial.readthedocs.io/en/latest/clipboard.html)
- [GTK4 Python API Reference](https://api.pygobject.gnome.org/)
- [GTK3 Python API Reference](https://lazka.github.io/pgi-docs/)

On Ubuntu desktop, some of these may be available by default but [can be installed with](https://pygobject.gnome.org/getting_started.html#ubuntu-getting-started):

```bash
sudo apt install python3-gi python3-gi-cairo gir1.2-gtk-4.0
```

This is why you might not see `gi` listed under `python3 -m pip list`.


## GTK4 First Attempt

This first proof of concept [borrows lines from the example in the documentation](https://pygobject.gnome.org/tutorials/gtk4/clipboard.html#example) and was tested on an Ubuntu 22.04 VM running GNOME + Wayland, from a python interpreter using GTK 4.0:

```python3
import gi

gi.require_version('Gdk', '4.0')
gi.require_version('Gtk', '4.0')

from gi.repository import Gdk, Gtk

clipboard = Gdk.Display.get_default().get_clipboard()
clipboard.read_text_async(None, None)

(process:012345): Gdk-CRITICAL **: 01:23:45.678: gdk_clipboard_read_text_async: assertion 'callback != NULL' failed
# SNIP
```

At this point (and there were other attempts that went nowhere) it's clear this will require more than just importing the library and attempting to read from one of the clipboards. [GTK 4.0 is event driven, operating on what are called signals and callbacks](https://pygobject.gnome.org/tutorials/gtk4/basics.html). Searching through both Google and ChatGPT returned references to both GTK4 and GTK3, with GPT-4o mentioning GTK3 will be simpler to work with to demo out this idea.


## GTK3 Second Attempt

Using GTK 3.0 is much easier, extracting only what we need [from an example similar to the one we just referenced in the GTK4 docs](https://python-gtk-3-tutorial.readthedocs.io/en/latest/clipboard.html), led to a working block of code:

```python3
# A nearly identical snippet was also suggested by GPT-4o, with minimal differences
import gi

gi.require_version("Gtk", "3.0")
from gi.repository import Gtk, Gdk

clipboard = Gtk.Clipboard.get(Gdk.SELECTION_CLIPBOARD)

# Set text manually to test and verify this works at all
# clipboard.set_text('This is a test.', -1)

text = clipboard.wait_for_text()

print(text)

```

On Ubuntu 22.04 running Wayland, this resulted in nothing being printed *until manually setting the clipboard via this interpreter session by using `clipboard.set_text('This is a test.', -1)`*. It's worth noting in those conditions (meaning Wayland) this doesn't suddenly set the clipboard for the desktop user either, it **appears to be local to the interpreter session, in the terminal**.

Now what happens when we try this on Kali running XFCE + X11?

```bash
└─$ python3
Python 3.13.3 (main, Apr 10 2025, 21:38:51) [GCC 14.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import gi
...
... gi.require_version("Gtk", "3.0")
... from gi.repository import Gtk, Gdk
...
... clipboard = Gtk.Clipboard.get(Gdk.SELECTION_CLIPBOARD)
...
>>> text = clipboard.wait_for_text()  # Paste this from our clipboard
>>> print(text)
text = clipboard.wait_for_text()      # This gets printed now, because it was in our clipboard
>>>                                   # Here, load a secret into your clipboard
>>> text = clipboard.wait_for_text()  # Type or up arrow to enter this again, updating the clipboard
>>>
>>> print(text)
Passw0rd!                             # This successfully retrieves it

```

It turns out we can read the clipboard!


## GTK3 GUI Demo

!!! question "GUI Demo"

    So sticking with GTK3, can we adapt the [demo GUI application](https://python-gtk-3-tutorial.readthedocs.io/en/latest/clipboard.html) to visually verify this and show what a real "windowed" application can do?

GPT-4o suggested using `GLib.timeout_add()` for the continuous "read" from the clipboard, which also seems to be [alluded to in this stackoverflow answer for refreshing a window, which instead calls `GLib.timeout_add_seconds()`](https://stackoverflow.com/questions/23817161/proper-way-force-refresh-of-window-in-gtk-3-using-pygobject).

```python
#!/usr/bin/env python3

# clipboard_snooper.py
#
# Tested on:
# - Ubuntu 22.04 (GNOME, Wayland)
# - Kali 2025.2 (XFCE, X11)
#
# Proof of concept to demo clipboard snooping on Linux.
# This was built using the example (linked below) in addition to
# conversing with GPT-4o which suggested using GLib + timeout_add()
# for the continuous "read" of the clipboard, and put together the
# update_clipboard_text() function as an added example.
#
# https://python-gtk-3-tutorial.readthedocs.io/en/latest/clipboard.html


import gi
gi.require_version("Gtk", "3.0")
from gi.repository import Gtk, Gdk, GLib


class ClipboardWindow(Gtk.Window):
    def __init__(self):
        super().__init__(title="Clipboard Snooping")

        grid = Gtk.Grid()

        self.clipboard = Gtk.Clipboard.get(Gdk.SELECTION_CLIPBOARD)
        self.entry = Gtk.Entry()
        # Removed the buttons here

        grid.add(self.entry)
        # Removed buttons here too

        self.add(grid)

        # This line sets the interval to try and read the clipboard once
        # the window is active, every 100ms does it almost immediately
        GLib.timeout_add(100, self.update_clipboard_text)

    # This function continuously reads from the clipboard
    def update_clipboard_text(self):
        text = self.clipboard.wait_for_text()
        if text is not None:
            self.entry.set_text(text)
        return True  # Causes this to execute continuously


win = ClipboardWindow()
win.connect("destroy", Gtk.main_quit)
win.show_all()
Gtk.main()

```

Save this script, and run it with:

```bash
python3 ./clipboard_snooping.py
```

This will open a small graphical window with a single text field, which should immediately become populated with your clipboard contents.


## Results

So how does this work in different scenarios?

| Dispaly | Context | Result |
| --- | --- | --- |
| **X11** | Inactive Window | ✅ Successfully read clipboard |
| **X11** | Terminal Session | ✅ Successfully read clipboard |
| **Wayland** | Inactive Window | ⚠️ Cannot read clipboard **until the window becomes active** |
| **Wayland** | Terminal Session | ❌ Cannot read clipboard **outside of session** |
| **Both** | Active Window | ✅ Successfully reads clipboard |

!!! tip "Reading the host clipboard from a VM"

    That last entry in the table for **both** even applies if you have an open, unlocked, VM window, where the last active application window within the VM is trying to read the clipboard contents. On an X11 host this also means the VM can steal clipboard contents silently in the background so long as that desktop session is still logged in, unlocked, and the app within the VM was left active.

Ultimately on Wayland, these specific methods and attempts were ***not*** able to steal anything from the clipboard without some user interaction. This is likely due to [Wayland's security architecture](https://en.wikipedia.org/wiki/Wayland_(protocol)#Differences_between_Wayland_and_X), so more research is needed to identify exactly how it's doing the isolation.


## Detection

!!! warning "Under Construction"

    This section in particular is still a work-in-progress, and will be updated over time.

- Both, the [SOCFortress Team's SysmonForLinux rules](https://github.com/socfortress/Wazuh-Rules/tree/main/Sysmon%20Linux) and [Neo23x0's auditd rules](https://github.com/Neo23x0/auditd) only see up to the execution of the source application, in this case python3
- Neither [`[sudo] busctl monitor` or `[sudo] dbus-monitor --session`](https://github.com/HackTricks-wiki/hacktricks/blob/master/src/linux-hardening/privilege-escalation/d-bus-enumeration-and-command-injection-privilege-escalation.md) worked to reveal this activity
- [pspy](https://github.com/DominicBreuker/pspy)?