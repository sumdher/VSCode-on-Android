# Visual Studio Code on Android + Remote Development via SSH (No root)

Guide to run Visual Studio Code and/or VSCodium on Android to remotely run code on a remote machine via SSH.

**Android** > **Termux** > (proot) **Ubuntu** > **VSCode**/**VSCodium**

While there are other methods like:
1. Cloud-based, frontend-only: [github.dev](https://github.dev/github/dev), [vscode.dev](https://vscode.dev/), [GitHub Codespaces](https://github.com/codespaces),
2. Full backend, local execution: [code-server](https://github.com/coder/code-server) (on a proot distro), which opens in the browser,
3. An app on the Play Store [VScode for Android](https://play.google.com/store/apps/details?id=dev.environment.VScode_PaidR1&hl=en), which basically is a webapp, putting it in the same category as 2.

What they all have in common is that they run in the browser. That's alright, but you can't do certain things like connect to a remote machine via SSH. Certain extensions are just not supported by web-based VSCode and remote-ssh is one of them. 

This attempt to install desktop VSCode on Android and then SSH-ing into a remote machine started as a proof-of-concept, but during the process, I found that Microsoft had already solved this with the [Remote - Tunnels](https://marketplace.visualstudio.com/items?itemName=ms-vscode.remote-server) extension. 

With this, a client can connect to a remote machine with [VS Code Server](https://code.visualstudio.com/docs/remote/vscode-server) installed, via a secure tunnel (without using SSH!). The client can run code remotely, make changes on the remote machine in any browser that supports web apps like vscode.dev or GitHub codespaces, etc., making the whole process platform-independent.

This **Remote - Tunnel** option is far better than running desktop VSCode on Ubuntu proot-ed in Termux on Android because of a couple of reasons: it’s much simpler, has far less latency compared to using a VNC desktop viewer for proot Ubuntu, lower power consumption,  and most importantly, because it's cool as 🗿. 

[Here](https://www.youtube.com/watch?v=SyLHXdXhE1U) is a quick video by VSCode devs to get you started with **Remote - Tunnel** if you want to try it. More info [here](https://code.visualstudio.com/docs/remote/tunnels).

---

Nonetheless, here's the guide to install the desktop VSCode/VSCodium version and then SSH into a remote desktop, because just because it _can_ be done, it means it _should_ be done, right?

After following this guide, you will be able to use the full-blown desktop app with all the desktop extensions and functionalities.

## Prerequisites

- [**Termux**](https://f-droid.org/en/packages/com.termux/) from [**F-Droid**](https://f-droid.org/en/), or from its [official page](https://termux.dev/en/) (not from the Play Store).
- [**Andronix**](https://play.google.com/store/apps/details?id=studio.com.techriz.andronix&hl=en) from Play Store.
- any **VNC viewer** from Play Store like [**RealVNC**](https://play.google.com/store/apps/details?id=com.realvnc.viewer.android&hl=en).
- About €2 in your bank account to purchase the modded distro in Andronix.

> **Case for paid distro:** Although Andronix offers several free distros, the modded OS versions include preinstalled VSCode (and other applications). I attempted a manual installation of VSCode on free distros, but it failed due to specific patching requirements. (Spoiler: even the modded OS had issues with VSCode launching, so below I share the patches I tried.)

## Good-to-haves:

 **Either:**
 - A **keyboard**, a **mouse** and an **external display**
 - Or, a **laptop** + **Samsung DeX** on both the latptop and the Android device

---

## Fixing Android 12+ killing forked child processes
On Android >=**12**, the maximum number of forked child processes an app can spawn is limited to 32. To prevent Android from killing these processes, you must increase this limit.
- The fix is [here](https://docs.andronix.app/android-12/andronix-on-android-12-and-beyond), thanks to Andronix devs.
- It's pretty verbose and exhaustive, so just follow the steps for your specific case and you should be good. 🎉

## Update Repos and Packages in Termux

Open Termux and run:
   ```bash
   termux-change-repo
   ```
Select "Single Mirror" and choose your region.
Update packages:

   ```bash
   pkg update && \
   pkg upgrade
   ```

---

## Install the Ubuntu distro

- In **Andronix**:
  - Navigate to "**Andronix Modded OS**" and select "**Ubuntu XFCE**".
  - Complete the purchase to receive a download link with keys. The installation command will be automatically copied to the clipboard.
- In **Termux**:
  - Paste the copied command and it will install the distro in the pwd.
  - Follow the on-screen instructions.
    > **Note:** The password you set for your user here during installation will also serve as your VNC password.

## Launching Ubuntu

- In Termux, run the bash file that the above command generates (in the same directory):

   ```bash
   ./start-andronix.sh
   ```
Now you should be logged in as your user in Ubuntu.

- Do the routine:
   ```bash
   sudo apt update && \
   sudo apt upgrade
   ```
- Start the VNC Server:
   ```bash
   vncserver-start
   ```
Select your desired resolution and it will start it at `localhost:1`.

### In your preferred VNC Viewer app: 

- Set the _address_ to `localhost:1`, and optionally give it a name.
- Enter your user password to access it (the one you set during Ubuntu installation).
- You should now see a Ubuntu desktop.
  > **Tip:** If the display quality is poor or some icons do not load properly, **set** the **picture quality** to **high**.
- When finished, stop the VNC server by running in the Ubuntu shell:
  ```bash
   vncserver-stop
   ```
- More details [here](https://docs.andronix.app/vnc/vnc-basics).

---

## Also, you might want to set up: python, pip, etc.

If not, skip directly to the [problems & fixes](#Problems-and-Fixes). 

### Run these in a terminal:

- Add a new repository to manage Python installations:
   ```bash
   sudo apt install -y software-properties-common
   ```
- Add the Python PPA:
   ```bash
   sudo add-apt-repository ppa:deadsnakes/ppa && \
   sudo apt update && \
   sudo apt install python3.11
   ```
-  If you want to create asymbolic link so that `python` points to `python3`:
   ```bash
   sudo ln -sf /usr/bin/python3 /usr/bin/python
   ```
-  Install `pip` and optionally, `pythonX.XX-venv`:
   ```bash
   sudo apt install python3-pip && \
   sudo apt install python3.11-venv
   ```
- Install Jupyter if you want to work with notebooks:
   ```bash
   sudo apt install jupyter ipykernel
   ```

---

# Problems and Fixes

After launching the Ubuntu GUI (via `./start-andronix.sh` and `vncserver-start` and connecting to  `localhos:1`), you may notice that while many preinstalled applications work, some do not. In particular, Firefox and VSCode may require fixes.

## Fixing Firefox

**The problem:** The tabs crash with Error: `Gah! Your tab just crashed!`, nothing can be opened.


**The fix:** Found it [here](https://www.youtube.com/watch?v=_SIGynLjrjk).

- Open Firefox and type `about:config` in the address bar and press enter.
- Click "Accept the risk and Continue".
- In the search preferences bar, type `sandbox` and,
   - Change `media.cubeb-sandbox` from `true` to `false`.
   - Change `security.sandbox.content.level` from `<num>` to `0`.

Restart Firefox. It should now work correctly. 🎉

---

> **Note:** From here on, run all the commands in the Ubuntu terminal, not Termux.

## Fixing VSCode/VSCodium

If you want to use **VSCodium** instead of VSCode, download from the [official source](https://github.com/VSCodium/vscodium/releases) and install it.
The problems they have are identical, and so are the fixes. They are exactly the same. 
The difference? Anywhere you see `code`, replace it with `codium` in **all** the commands below:

**The problem:** It doesn't launch. Many issues.

### Step 1: Verify if it launches

- **Graphical launch**: Search for VSCode (VSCodium) in the start menu and click it.
  > **Note:** It doesn't launch in my device (Samsung S24 Ultra) but it looks like the problem is general to proot installations.

   If it opens, skip to the [next section](#Setting-up-Python-and-pip).
- **Terminal Launch**: Run
  ```bash
  code
  # or `codium`; be mindful of this from here on...
  ```

  If it opens, skip to the [next section](#Setting-up-Python-and-pip).

### Step 2: Resolve the `/proc/version: Permission denied` Error

If you are still reading this, VSCode did not launch (it should only take about 2-3 seconds).

When you run `code`, you probably saw an error like:
   ```perl
   grep: /proc/version: Permission denied
   ```

   This occurs because `/usr/bin/code` is trying to access `/proc/version` for a WSL installation check, which is a protected file in Android.

**The fix:** Comment it out

- Install your preferred text editor, (e.g., `gedit`):
  ```bash
  sudo apt install gedit
  ```
- Edit the `code` file (you can confirm the path by running `which code`):
  ```bash
  gedit /usr/bin/code
  ```
- Locate this `if` block:
  ```bash
  if grep -qi Microsoft /proc/version && [ -z "$DONT_PROMPT_WSL_INSTALL" ]; then
  ```
   Comment out the entire block up to the corresponding `fi`; either by adding `#` at the start of each line or enclosing the block in a shell comment (start with `: '` and end with `'` .

  Like:
```bash
: '
if grep -qi Microsoft /proc/version && [ -z "$DONT_PROMPT_WSL_INSTALL" ]; then
        echo To use Visual Studio Code with the Windows Subsystem for Linux, please install Visual Studio Code in Windows and uninstall the Linux version in WSL. You can then use the `code` command in a WSL terminal just as you would in a normal command prompt. 1>&2
        printf Do you want to continue anyway? [y/N]  1>&2
        read -r YN
        YN=
        case  in
                y | yes )
                ;;
                * )
                        exit 1
                ;;
        esac
        echo To no longer see this prompt, start Visual Studio Code with the environment variable DONT_PROMPT_WSL_INSTALL defined. 1>&2
fi
'
```

After saving the changes, run `code` again, the `/proc/version: Permission denied` should be gone. 🎉

### Step 3 Fixing the SUID Sandbox Error

If VSCode still doesn’t launch, run:
```bash
code --verbose
```
You may see an error like:

```diff
- [10548:0218/173116.917214:FATAL:setuid_sandbox_host.cc(163)]
The SUID sandbox helper binary was found, but is not configured correctly.
Rather than run without sandboxing I'm aborting now.
+ You need to make sure that /usr/share/code/chrome-sandbox is owned by root and has mode 4755.
```

**The fix:**

- In terminal, run these 2 commands:
   ```bash
   sudo chown root:root /usr/share/code/chrome-sandbox && \
   sudo chmod 4755 /usr/share/code/chrome-sandbox
   ```
   Replace the `/usr/share/code/chrome-sandbox` with the path in the error message of `code --verbose`, if it is different.

This should essentially fix it. Try running `code` again after reloading Ubuntu (which I didn't do, so I ran the following commands regardless). 

If it launches after this, skip to the [next section](#Setting-up-Python-and-pip).

- If it still doesn't launch, run:
   ```bash
   echo 'export DONT_PROMT_WSL_INSTALL=true' >> ~/.bashrc && \
   echo 'export ELECTRON_DISABLE_SANDBOX=1' >> ~/.bashrc && \
   source ~/.bashrc
   ```

Now when you run `code` (`codium`), it should successfully launch **VSCode** (**VSCodium**) on your Android device. 🦾🎉

> **Tip:** Sometimes, launching VSCode by clicking its icon in the start menu or desktop icon does not work.
> In that case, right click on the icon and select "Edit launcher". Set `command` to  `/usr/bin/code` (or wherever the path of `code` is).
> Now, it's good to go 🎉

In **VSCode**/**VSCodium**, install any necessary packages as per your need like languages, Jupyter, remote-ssh, etc.

---

## Fixing Jupyter Kernel Load error

When working with Python notebooks in VSCode, you might encounter an error like:
```diff
- Failed to start the Kernel. 
/home/<user>/<workspace>/<py_venv_dir>/lib/python3.11/site-packages/jupyter_client/localinterfaces.py:56: 
- UserWarning: Unexpected error discovering local network interfaces: 
- [Errno 13] Permission denied ret = f(**kwargs) Permission denied (src/ip_resolver.cpp:542). 
+ View Jupyter log for further details.
```
This is a known issue with proot.

**The fix**:
Found it [here](https://github.com/termux/proot/issues/248).

Summarizing,
- Run in terminal:
   ```bash
   mkdir -p ~/jupyter_fix && \
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
  >**Note:** If you are using a virtual environment, make sure `jupyter` and `ipywidgets` are installed or else, the output won't be shown.

---

# SSH-ing into a Remote Computer via VSCode/VSCodium for Remote development

Out of the box, there are no major issues with this. But there was something that needed to be fixed. Read on...

**Context:** I had SSH keys setup in Ubuntu (not Termux Android). I had a connection to a private VPN (using [OpenVPN](https://play.google.com/store/apps/details?id=net.openvpn.openvpn&hl=en) app (on Android)).

I had also installed the necessry SSH extensions in VSCode. 

**The problem:**
- It successfully connected SSH-ically to the remote computer, but it **fails to recognize existing kernels and virtual envs** in the remote computer.

**The fix:**
- After SSH-ing into the remote computer in VSCode, in the workspace directory, make a new directory `.vscode` and create a file in it `settings.json`.
- In `settings.json`, add these lines:

  > **Note:** Replace the path and version with the actual path of `venv` and its python version in the remote computer)
  ```json
  {
      "python.defaultInterpreterPath":"<path>/<to>/your>/<venv>/bin/python<X>"
      "python.analysis.extraPaths": ["<path>/<to>/your>/<venv>/lib/python<X.XX>/site-packages"]
  }
  ```
- Save it and restart VSCode. Now, everything should work well. 🎉


Happy coding! 🦾

ఇంక సెలవు 🚀

---

# Other apps

Andronix patched some useful softwares: [browsers](https://docs.andronix.app/software/browsers), [GIMP](https://docs.andronix.app/software/gimp), [IDEs](https://docs.andronix.app/software/IDEs) and [Libre Office](https://docs.andronix.app/software/libre-office).

>**Note:** Before fixing this, I reinstalled VSCode from _IDEs'_ page. It had thrown a new error: `Illegal instruction` which I believe has something to do with the CPU architecture of my device: Samsung S24 ultra. So, back up the installed VSCode before reinstalling it.
