# VSCode in Android

Android > Termux > (proot) Ubuntu > VSCode

## Prerequisites

- [**Termux**](https://f-droid.org/en/packages/com.termux/) from [**F-Droid**](https://f-droid.org/en/) (not from Play Store), or from its [official page](https://termux.dev/en/).
- [**Andronix**](https://play.google.com/store/apps/details?id=studio.com.techriz.andronix&hl=en) from Play Store
- any **VNC viewer** from Play Store like [**RealVNC**](https://play.google.com/store/apps/details?id=com.realvnc.viewer.android&hl=en)
- about 2€ in your bank account (for the modded distro in Andronix)

Case for paid distro: Although there are a bunch of free distros in Andronix, the modded OSs come with preinstalled VSCode (and other applications). I tried installng them manually but it failed. Needs some specific pathching and they did it. 
Spoiler: It also didn't work in the Modded OS. VSCode just did not open and there were some other issues. Here, I will share the patches I tried to make it work.

## Update repo and packages in Termux
   
1. Go to [GitHub](https://github.com) and sign in **Tokens (classic)**
2. Enter this in Termux:
   ```bash
   termux-change-repo
   ```
Select "single" and select your region
Update packages by:

   ```bash
   pkg update
   pkg upgrade
   ```
## Install the Ubuntu distro

- In **Andronix**
  - Navigate to "Modded OS" and select "Ubuntu". Pay and you will get the download link which you paste and run in Termux.
  - Follow the on screen instructions and it will be installed.

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

## VSCode
- Finally, there is a front-end **React server**.


### Main server

1. **Navigate to the Server Directory**
   ```bash
   cd main_server
   ```

2. **Set up Virtual Environment**
   ```bash
   python3 -m venv venv
   ```
   
   ```bash
   .\venv\Scripts\activate  # For macOS/Linux: `source venv/bin/activate`
   ```

3. **Install Dependencies**
   ```bash
   pip install -r requirements.txt
   ```

### Similarly, **Data server**

   ```bash
   cd data_server
   python3 -m venv venv
   .\venv\Scripts\activate  # For macOS/Linux: `source venv/bin/activate`
   pip install -r requirements.txt
   ```

### React Server

1. **Navigate to the Client Directory**
      
   ```bash
   cd client
   ```

2. **Install Dependencies**
   ```bash
   npm install
   ```

 ## Run the StopTracker Webapp

   Open 3 different terminals and run the following commands:

   Terminal 1 (**/main_server**):
   ```bash
   python3 main_server.py
   ```

   Terminal 2 (**/data_server**):
   ```bash
   python3 data_server.py
   ```
   If the **main server** is on another machine, pass the IP address of the main server:
   ```bash
   python3 data_server.py <local:ip:of:mainserver>
   ```
   Terminal 3 (**/client**):

   ```bash
   npm run dev
   ```

5. **In any browser, open the link:**
    ### `http://<local:ip:of:reactserver>:5173/`



## Start the webapp

1. Press the button "Go Live!" to visualize trajectories and stops in real time.

2. To change the layout of the tables/POIs, upload `.geojson` files by clicking "Update Map" button in the header.

3. Download a structured file (.csv) of the detected stops using the Download button in the header.


## Stop the webapp

1. Terminate (by pressing CTRL^C) in this sequence: the data_server, main_server and then the React client in their respective terminals.



</br></br></br></br></br></br></br></br></br></br></br></br></br></br></br></br></br></br>
Developped by: [Sudheer](https://github.com/sumdher) and [Fatima Hachem](https://github.com/HachemFatima)
