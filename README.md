# Visual Studio Code on Android

Guide to run VSCode on Android.

Android > Termux > (proot) Ubuntu > VSCode

## Prerequisites

- [**Termux**](https://f-droid.org/en/packages/com.termux/) from [**F-Droid**](https://f-droid.org/en/), or from its [official page](https://termux.dev/en/) (not from the Play Store).
- [**Andronix**](https://play.google.com/store/apps/details?id=studio.com.techriz.andronix&hl=en) from Play Store.
- any **VNC viewer** from Play Store like [**RealVNC**](https://play.google.com/store/apps/details?id=com.realvnc.viewer.android&hl=en).
- Approximately €2 in your bank account to purchase the modded distro in Andronix.

> **Case for paid distro:** Although Andronix offers several free distros, the modded OS versions include preinstalled VSCode (and other applications). I attempted a manual installation of VSCode on free distros, but it failed due to specific patching requirements. (Spoiler: even the modded OS had issues with VSCode launching, so below I share the patches I tried.)

## Good-to-haves:

**Either:**
 - A keyboard, a mouse and an external display          **or**
 - a laptop + _Samsung DeX_ on both the devices
---

## Fixing Android 12 killing forked child processes
On Android 12 or later, the maximum number of forked child processes an app can spawn is limited to 32. To prevent Android from killing these processes, you must increase this limit.
- The fix is [here](https://docs.andronix.app/android-12/andronix-on-android-12-and-beyond), thanks to Andronix devs.
- It's pretty verbose and detailed so just follow the steps for your specific case and you should be good. 🎉

## Update Repos and Packages in Termux

Open Termux and run:
   ```bash
   termux-change-repo
   ```
Select "Single Mirror" and choose your region.
Update packages:

   ```bash
   pkg update
   pkg upgrade
   ```

---

## Install the Ubuntu distro

- In **Andronix**:
  - Navigate to "**Andronix Modded OS**" and select "**Ubuntu XFCE**".
  - Complete the purchase to receive a download link and keys. The installation command will be automatically copied to your clipboard.
- In **Termux**:
  - Paste the copied command and it will install the distro in the pwd.
  - Follow the on screen instructions.
    > **Note:** The password you set for your user here during installation will also serve as your VNC password.

## Launching Ubuntu

- In Termux, run the bash file that the above command generates (in the same directory):

   ```bash
   ./start-andronix.sh
   ```
Now you should be logged in as your user in Ubuntu.

- Do the routine:
   ```bash
   sudo apt update
   sudo apt upgrade
   ```
- Start the VNC Server:
   ```bash
   vncserver-start
   ```
Select your desired resolution and it will start it at `localhost:1`.

- Open your VNC Viewer: 

- Set the _address_ to `localhost:1`, and optionally give it a name.
- Enter your user password to access it (the one you set during Ubuntu installation).
- You should now see a Ubuntu desktop.
  > **Tip:** If the display quality is poor or some icons do not load properly, adjust the picture quality to high.
- When finished, stop the VNC server by running in the Ubuntu shell:
  ```bash
   vncserver-stop
   ```
- More details [here](https://docs.andronix.app/vnc/vnc-basics).

---

# Problems and Fixes

After launching the Ubuntu GUI (via `./start-andronix.sh` and `vncserver-start` and connecting to  `localhos:1` you may notice that while many preinstalled applications work, some do not. In particular, Firefox and VSCode may require fixes.

## Fixing Firefox

**The problem:** The tabs crash, nothing can be opened.

**The fix:**

- Open Firefox and type `about:config` in the address bar and press enter.
- Click "Accept the risk and Continue".
- In the search preferences bar, type `sandbox` and,
   - Change `media.cubeb-sandbox` from `true` to `false`.
   - Change `security.sandbox.content.level` from `<num>` to `0`.
- Restart Firefox. It should now work correctly. 🎉

---

> **Note:** From here on, run all the commands in the Ubuntu terminal, not Termux.

## Fixing VSCode

**The problem:** It doesn't launch. Many issues.

### Step 1: Verify if VSCode launches

- Graphical Launch: Search for VSCode in the start menu and click it.
  > **Note:** It doesn't launch in my device (Samsung S24 Ultra) but it looks like the problem is general to proot implementations like this.

   If it opens, skip to the next section.
- Terminal Launch: Run
  ```bash
  code
  ```

  If it opens, skip to the next section.

### Step 2: Resolve the `/proc/version: Permission denied` Error

If you are still reading this, VSCode did not launch (it should only take about 2-3 seconds).

When you run `code`, you probably saw an error like:
   ```perl
   grep: /proc/version: Permission denied
   ```

   This occurs because `/usr/bin/code` is trying to access `/proc/version` for WSL installation check, which is a protected file in Android.

**The fix:** Comment it out

- Install your preferred text editor, (e.g., `gedit`):
  ```bash
  sudp apt install gedit
  ```
- Open `code` (you can confirm the path with `which code`):
  ```bash
  gedit /usr/bin/code
  ```
- Locate this `if` block:
  ```bash
  if grep -qi Microsoft /proc/version && [ -z "$DONT_PROMPT_WSL_INSTALL" ];
  ```
   Comment out the entire block up to the corresponding `fi`; either by adding `#` at the start of each line or enclosing the block in a shell comment (start with `: '` and end with `'`).

After saving the changes, run `code` again, the `/proc/version: Permission denied` should be gone. 🎉

### Step 3 Fixing the SUID Sandbox Error

If VSCode still doesn’t launch, run:
```bash
code --verbose
```
You may see an error like:

```vbnet
[10548:0218/173116.917214:FATAL:setuid_sandbox_host.cc(163)]
The SUID sandbox helper binary was found, but is not configured correctly.
Rather than run without sandboxing I'm aborting now.
You need to make sure that /usr/share/code/chrome-sandbox is owned by root and has mode 4755.
```

**The fix:**

- In terminal, run these 2 commands: (replace the path with the error message of `code --verbose` if it is different)
   ```bash
   sudo chown root:root /usr/share/code/chrome-sandbox
   sudo chmod 4755 /usr/share/code/chrome-sandbox
   ```

This should essentially fix it. Try running `code` again after reloading Ubuntu (which I didn't do so I ran the following commands). If it launches, skip to the next section.

- If it still doesn't launch, run:
   ```bash
   echo 'export DONT_PROMT_WSL_INSTALL=true' >> ~/.bashrc
   echo 'export ELECTRON_DISABLE_SANDBOX=1' >> ~/.bashrc
   source ~/.bashrc
   ```

Now when you run `code`, it should successfully launch VSCode. 🎉

---

## Setting up Python and pip

**No problems here**

In VSCode, install any necessary packages as per your need.

Also, you might want to install `python<X.XX>`, `python<X.XX>-venv`(optional) and `pip` globally with:

- Add a new repository to manage Python installations. Run in terminal:
   ```bash
   sudo apt install -y software-properties-common
   ```
- Add the Python PPA:
   ```bash
   sudo add-apt-repository ppa:deadsnakes/ppa
   sudo apt update
   sudo apt install python3.11
   ```
-  If you want to create asymbolic link so that `python` points to `python3`:
   ```bash
   sudo ln -sf /usr/bin/python3 /usr/bin/python
   ```
-  Install `pip` and optionally, `pythonX.XX-venv`:
   ```bash
   sudo apt install python3-pip
   sudo apt install python3.11-venv
   ```
- Install Jupyter if you want to work with notebooks:
   ```bash
   sudo apt install jupyter ipykernel
   ```
---

---

## Fixing Jupyter Kernel Load error

When working with Python notebooks in VSCode, you might encounter an error like:
```bash
Failed to start the Kernel. 
/home/<user>/<workspace>/<python_dir>/lib/python3.11/site-packages/jupyter_client/localinterfaces.py:56: 
UserWarning: Unexpected error discovering local network interfaces: 
[Errno 13] Permission denied ret = f(**kwargs) Permission denied (src/ip_resolver.cpp:542). 
View Jupyter log for further details.
```
This is a known issue with proot.

**The fix**:
Found it [here](https://github.com/termux/proot/issues/248).

Summarizing,
- Run in terminal:
   ```bash
   mkdir -p ~/jupyter_fix
   touch ~/jupyter_fix/skip_getifaddrs.c
   gedit ~/jupyter_fix/skip_getifaddrs.c
   ```
- Paste the following in `~/jupyter_fix/skip_getifaddrs.c`:
   ```c
   #include <errno.h>
   #include <ifaddrs.h>
   
   int getifaddrs(struct ifaddrs **ifap) {
       errno = EOPNOTSUPP;
       return -1;
   }
   ```
-  Compile it as a shared library
   ```bash
   gcc -shared -fPIC -o ~/jupyter_fix/skip_getifaddrs.so ~/jupyter_fix/skip_getifaddrs.c
   ```
- Navigate to VSCode
     - Press `Ctrl+Shift+P`, look for `Preferences: Open Settings (UI)`.
     - Search for "env file", look for `Python: Env File`.
     - Ensure the path is `${workspaceFolder}/.env`. If not, make it so.
- In the workspace folder of VSCode, create (or edit) a file named `.env`, and add the line:
     ```bash
     LD_PRELOAD=~/jupyter_fix/skip_getifaddrs.so
     ```
- Restart VSCode and now the kernel should start without any errors. 🎉
  >**Note:** If you are using a virtual environment, make sure `jupyter` and `ipywidgets` is installed or else, the output won't be shown.

---

# SSH-ing to a Remote Computer from Ubuntu via VSCode for Remote development.

Out of the box, there are no major issues with this. But there was something that needed to be fixed. Read on...

**Context:** I had SSH keys setup in Ubuntu (not Termux Android). I had a connection to a private VPN (using [OpenVPN](https://play.google.com/store/apps/details?id=net.openvpn.openvpn&hl=en) app (on Android)).

I had also installed the necessry SSH extensions in VSCode. 

**The problem:**
- It successfully connects SSH-ically to remote computers, but it **fails to recognize existing kernels and virtual envs** in the remote compputers.

**The fix:**
- After connecting to the remote coumputer in VSCode, in your workspace directory, make a directory `.vscode` and create a file in it `settings.json`.
- In `settings.json`, add these lines:
  ```json
  {
      "python.defaultInterpreterPath":"<path>/<to>/your>/<venv>/bin/pythonX"
      "python.analysis.extraPaths": ["<path>/<to>/your>/<venv>/lib/pythonX.XX/site-packages"]
  }
  ```
- Save it and restart VSCode. Now, everything should work well. 🎉


Happy coding! Inka selavu 

---

# Other apps

Andronix patched some useful softwares: [browsers](https://docs.andronix.app/software/browsers), [GIMP](https://docs.andronix.app/software/gimp), [IDEs](https://docs.andronix.app/software/IDEs) and [Libre Office](https://docs.andronix.app/software/libre-office).

>**Note:** Before fixing this, I reinstalled VSCode from _IDEs'_ page. It had thrown a new error: `Illegal instruction` which I believe has something to do with the CPU architecture of my device: Samsung S24 ultra. So, back up the installed VSCode before reinstalling it.
