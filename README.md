# VSCode in Android

Android > Termux > (proot) Ubuntu > VSCode

## Prerequisites

- [**Termux**](https://f-droid.org/en/packages/com.termux/) from [**F-Droid**](https://f-droid.org/en/) (not from Play Store), or from its [official page](https://termux.dev/en/).
- [**Andronix**](https://play.google.com/store/apps/details?id=studio.com.techriz.andronix&hl=en) from Play Store.
- any **VNC viewer** from Play Store like [**RealVNC**](https://play.google.com/store/apps/details?id=com.realvnc.viewer.android&hl=en).
- about 2€ in your bank account (for the modded distro in Andronix).

Case for paid distro: Although there are a bunch of free distros in Andronix, the modded OSs come with preinstalled VSCode (and other applications). I tried installng them manually but it failed. Needs some specific pathching and they did it. 
Spoiler: It also didn't work in the Modded OS. VSCode just did not open and there were some other issues. Here, I will share the patches I tried to make it work.

## Fixing Android 12 killing forked child processes
Before anything, if you're using Android >=12, the maximum number of forked child processes that can be started by apps needs to be increaed from 32 to a big number. If this isn't done, Android just kills them. 
- The fix is [here](https://docs.andronix.app/android-12/andronix-on-android-12-and-beyond), thanks to Andronix devs.
- It's pretty verbose and detailed so just follow the steps for your specific case and you should be good.

## Update repo and packages in Termux

Enter this in Termux:
   ```bash
   termux-change-repo
   ```
Select "Single Mirror" and select your region.
Update packages by:

   ```bash
   pkg update
   pkg upgrade
   ```
## Install the Ubuntu distro

- In **Andronix**
  - Navigate to "**Andronix Modded OS**" and select "**Ubuntu XFCE**".
  - Pay and you will get the download link with keys to the distro. It will automatically copy a command to the clipboard.
  - Open Termux and paste the command that will install the distro in the pwd.
  - Follow the on screen instructions and it will be installed (note: the password you set for your user will be the VNC password).

## Launch Ubuntu

Run the bash file that the above command generates (in the same directory) in Termux:

   ```bash
   ./start-andronix.sh
   ```
Now you should be logged in as your user in Ubuntu.

Do the routine:
   ```bash
   sudo apt update
   sudo apt upgrade
   ```
Start VNC Server:
   ```bash
   vncserver-start
   ```
Select your desired resolution and it will start it at `localhost:1`.

Open a **VNC Viewer** 

- Set it up by seting `localhost:1` for _Address_, and optionally a name.
- Enter your user password to access it (the one you set during Ubuntu installation).
- You should now see a GUI Ubuntu desktop. (You might want to change the _picture quality_ to _high_ if it looks bad. Sometimes, not all icons load properly).
- And make sure to stop the vncserver after you are done (in the Termux Ubuntu shell where you logged in using `./start-andronx.sh`) by running:
  ```bash
   vncserver-stop
   ```
- More details [here](https://docs.andronix.app/vnc/vnc-basics).

## VSCode
- Finally, there is a front-end **React server**.

# Fixing VSCode, Firefox...

Launch GUI desktop of Ubuntu from Termux by running `./start-andronix.sh` and `vncserver-start` and finally using a vnc viewer to connect to `localhost:1`.

Take a stroll, you will see many applications preinstalled. Some of them work, some don't. VSCode is one of them. Firefox opens, but the tabs crash. 

## Fixing Firefox
- Type `about:config` in the address bar and hit enter. Press the button "Accept the risk and Continue".
- In the search preferences bar, type `sandbox`.
- Change `media.cubeb-sandbox` from `true` to `false`.
- Change `security.sandbox.content.level` from `<num>` to `0`.
- Reopen Firefox and it should work now.

## Fixing VSCode
- First of all, see if it runs well. It didn't work in my phone (Samsung S24 Ultra) but it looks like the problem is general to proot implementations like this.
     - In "start" menu, search for VSCode and try clicking on it. If it opens, skip to the next section.
     - In the terminal, run `code`. If it opens, skip to the next section.
- If you are still reading this, VSCode did not launch (it only takes about 2-3 seconds).
   - When you run `code`, you probably saw an error: `grep: /proc/version: Permission denied`.
   - The reason is that `/usr/bin/code` needs to access `/proc/version` for WSL installation check. This needs to be commented out.
     - Install your preferred editor, I used _gedit_ because it has GUI. Install it `sudp apt install <gedit>/<nano>...`.
     - Open terminal and run `gedit /usr/bin/code` (or check get the correct path by running `which code`).
     - You will see an `if` block: `if grep -qi Microsoft /proc/version && [ -z "$DONT_PROMPT_WSL_INSTALL" ];`
     - Comment the whole block until `fi`. You can either add `#` before each line, or enclose the whole block within `: '` and `'`.

After doing this, running `code` again, you will no longer see the _/proc/version: Permission denied_ error.

But it still doesn't launch. Run `code --verbose` and you will see the issue.

You will see something like:

```
[10548:0218/173116.917214:FATAL:setuid_sandbox_host.cc(163)]
The SUID sandbox helper binary was found, but is not configured correctly.
Rather than run without sandboxing I'm aborting now.
You need to make sure that /usr/share/code/chrome-sandbox is owned by root and has mode 4755.
```

**The fix:**

In terminal, run these 2 commands: (replace the path with the error message of `code --verbose` if it is different)
```bash
sudo chown root:root /usr/share/code/chrome-sandbox
sudo chmod 4755 /usr/share/code/chrome-sandbox
```

This should essentially fix it. Try running `code` again after reloading Ubuntu (which I didn't do and it did not launch). If it launches, skip to the next section.
If it still doesn't launch, run these in the terminal:
```bash
echo 'export DONT_PROMT_WSL_INSTALL=true' >> ~/.bashrc
echo 'export ELECTRON_DISABLE_SANDBOX=1' >> ~/.bashrc
source ~/.bashrc
```

Now when you run `code`, it should launch successfully 🎉

## Setting up Python, pip

In VSCode, you install the necessary packages as per your need.

Also, you might want to install python<X.XX>, python<X.XX>-venv and pip globally with:

- Add a new repository to manage Python installations. Run in terminal:
   ```bash
   sudo apt install -y software-properties-common
   ```
- Add the Python repo:
   ```bash
   sudo add-apt-repository ppa:deadsnakes/ppa
   sudo apt update
   sudo apt install python3.11
   ```
-  If you want to create asymbolic link so that `python` points to `python3`:
   ```bash
   sudo ln -sf /usr/bin/python3 /usr/bin/python
   ```
-  Install pip and optionally, pythonX.XX-venv:
   ```bash
   sudo apt install python3-pip
   sudo apt install python3.11-venv
   ```

## Jupyter kernel error

If you want to work with Python notebooks, you might want to test it by creating a _.ipynb_ file.

Select the Python kernel and interpretor. Print something in the first cell or run some python script.
You will probably get an error like: 
```
Failed to start the Kernel. 
/home/<user>/<workspace>/<python_dir>/lib/python3.11/site-packages/jupyter_client/localinterfaces.py:56: 
UserWarning: Unexpected error discovering local network interfaces: 
[Errno 13] Permission denied ret = f(**kwargs) Permission denied (src/ip_resolver.cpp:542). 
View Jupyter log for further details.
```

**The fix**:
It is a proot issue. Found it [here](https://github.com/termux/proot/issues/248).

Summarizing,
- Run in terminal:
   ```bash
   touch skip_getifaddrs.c
   gedit skip_getifaddrs.c
   ```
- Paste the following:
```c
#include <errno.h>
#include <ifaddrs.h>

int getifaddrs(struct ifaddrs **ifap) {
    errno = EOPNOTSUPP;
    return -1;
}
```


