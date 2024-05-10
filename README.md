# 🏠 GPT Home 🤖💬

![Ubuntu Server Version](https://img.shields.io/badge/Ubuntu_Server-v23.04-orange?style=flat-square&logo=ubuntu)
![Raspberry Pi Version](https://img.shields.io/badge/Raspberry_Pi-4B-red?style=flat-square&logo=raspberry-pi)
![Python Version](https://img.shields.io/badge/Python-v3.11-blue?style=flat-square&logo=python)
![Node.js Version](https://img.shields.io/badge/Node.js-v18.17.1-green?style=flat-square&logo=node.js)
[![Release](https://img.shields.io/github/v/release/judahpaul16/gpt-home?style=flat-square)](https://github.com/judahpaul16/gpt-home/tags)

ChatGPT at home! Basically a better Google Nest Hub or Amazon Alexa home assistant. Built on the Raspberry Pi using the OpenAI API.

![My Build](screenshots/my_build.jpg)

This guide will explain how to build your own. It's pretty straight forward. You can also use this as a reference for building other projects on the Raspberry Pi.

* *In my testing, I am running Ubuntu Server 23.04 on a Raspberry Pi 4B. You may need to make certain modifications to accommodate other operating systems (e.g., the package manager `apt` used in the setup script is debian specific). Theoretically, the app should run on any linux system, but I can only vouch for the versions listed in the [compatibility table](#-compatibility).*

* *You can theoretically use any USB/3.5mm speaker or microphone.*

## 🚀 TL;DR
```bash
cd ~
echo "export OPENAI_API_KEY=\"your_openai_api_key\"" >> ~/.bashrc
source ~/.bashrc
wget -O setup.sh https://raw.githubusercontent.com/judahpaul16/gpt-home/main/contrib/setup.sh
sudo chown -R $(whoami):$(whoami) .
sudo chmod -R 755 .
sudo chmod +x setup.sh
./setup.sh
```

## 🔌 Integrations
- [x] OpenAI
- [x] Spotify
- [x] Philips Hue
- [x] OpenWeatherMap
- [ ] Open-Meteo
- [ ] Calendar
- [ ] Alarms & Reminders

## ⚠️ Schematics / Wiring Diagram
### Caution: Battery Connection
**IMPORTANT**: The image on the left is for illustration purposes. ***Do not connect the battery directly to the Raspberry Pi. Use a UPS or power supply with a battery like this [one](https://a.co/d/1rMMCPR).*** Connecting the battery directly to the Raspberry Pi can cause damage to the board from voltage fluctuations.

Before connecting the battery, ensure that the polarity is correct to avoid damage to your Raspberry Pi or other components. Disconnect power sources before making changes.

<table style="border-collapse: collapse; border: 0;">
  <tr>
    <td style="border: none;"><img src="screenshots/schematic_bb.png" alt="Schematics Breadboard" height="350px" /></td>
    <td style="border: none;"><img src="screenshots/schematic_schem.png" alt="Schematics Schematic" height="350px" /></td>
  </tr>
</table>
<span style="font-size: 1em; display:block;">[click to enlarge]</span>


---

## 🛠 My Parts List

### Core Components
- **Raspberry Pi 4B**: [Link](https://a.co/d/aH6YCXY) - $50-$70
- **Mini Speaker**: [Link](https://a.co/d/9bN8LZ2) - $18
- **128 GB MicroSD card**: [Link](https://a.co/d/0SxSg7O) - $13
- **USB 2.0 Mini Microphone**: [Link](https://a.co/d/eIrQUXC) - $8

---

### Optional Components
- **128x32 OLED Display**: [Link](https://a.co/d/4Scrfjq) - $13-$14
- **Standoff Spacer Column M3x40mm**: [Link](https://a.co/d/ees6oEA) - $14
- **M1.4 M1.7 M2 M2.5 M3 Screw Kit**: [Link](https://a.co/d/4XJwiBY) - $15
- **Raspberry Pi UPS Power Supply with Battery**: [Link](https://a.co/d/1rMMCPR) - $30
- **Cool Case for Raspberry Pi 4B**: [Link](https://a.co/d/idSKJIG) - $16

---

## 💲 Total Price Range
- **Core Components**: $102-$123
- **Optional Components**: $75
- **Total (Without Optional)**: $102-$123
- **Total (With Optional)**: $177-$198

---

## 📶 Configuring Wi-Fi via wpa_supplicant

To configure Wi-Fi on your Raspberry Pi, you'll need to edit the `wpa_supplicant.conf` file and ensure the wireless interface is enabled at boot.

1. Install `net-tools` to get the `ifconfig` command:
   ```bash
   sudo apt install net-tools
   ```

2. To enable the wireless interface (`wlan0` in most cases) at boot, add the following command to `/etc/rc.local` before the `exit 0` line:  
    *Create the file if it doesn't exist*
    ```bash
    sudo vim /etc/rc.local
    ```
    Add the following contents:
    ```bash
    #!/bin/bash
    sudo ifconfig wlan0 up &
    sudo wpa_supplicant -i wlan0 -c /etc/wpa_supplicant/wpa_supplicant.conf -B &
    sudo dhclient wlan0 &
    exit 0
    ```
    Ensure the file has executable permissions and is enabled as a service:
    ```bash
    sudo chmod +x /etc/rc.local
    sudo systemctl enable rc-local.service
    sudo systemctl start rc-local.service
    ```

3. Open the configuration file in a text editor:
    ```bash
    sudo vim /etc/wpa_supplicant/wpa_supplicant.conf
    ```

4. Add the following lines at the end of the file:  
*You can define multiple `network` blocks for multiple Wi-Fi networks*
    ```bash
    network={
        ssid="Your_Wi-Fi_Name"
        psk="Your_Wi-Fi_Password"
        key_mgmt=WPA-PSK
    }
    ```
    Replace `Your_Wi-Fi_Name` and `Your_Wi-Fi_Password` with your actual Wi-Fi credentials.

4. Ensure `wpa_supplicant` service starts at boot:
    ```bash
    sudo systemctl enable wpa_supplicant.service
    ```

5. Start `wpa_supplicant` service:
    ```bash
    sudo systemctl start wpa_supplicant.service
    ```

Your Raspberry Pi should now connect to the Wi-Fi network automatically on boot. If you face issues, refer to the [official Raspberry Pi documentation on wireless connectivity](https://www.raspberrypi.com/documentation/computers/configuration.html#setting-up-a-wireless-lan-via-the-command-line).

## 🛠 System Dependencies

Before running this project on your Raspberry Pi, you'll need to install some system-level dependencies in addition to the Python packages.

1. Synchoronize your system clock:
    ```bash
    sudo timedatectl set-ntp on
    ```

2. Update your package list:
    ```bash
    sudo apt update
    ```

2. Make sure the Universe repository is enabled:
    ```bash
    sudo add-apt-repository universe
    sudo apt update
    ```

### Installing Dependencies  
If you want to use the [setup.sh](#-example-setup-script) script, you can skip this section. Otherwise, you can install the dependencies manually.

1. **OpenAI API Key**: Required for OpenAI's GPT API.  
    Setup: Set up as an environment variable.  

2. **Python 3.11.x**: Required for running the Python code.  
   Installation: `sudo apt-get install -y python3.11 python3-dev`

3. **PortAudio**: Required for `pyttsx3` (text-to-speech).  
   Installation: `sudo apt-get install -y portaudio19-dev`

4. **ALSA Utilities**: Required for audio configuration.  
   Installation: `sudo apt-get install -y alsa-utils`

5. **JPEG Library**: Required for Pillow.  
   Installation: `sudo apt-get install -y libjpeg-dev`

6. **Build Essentials**: Required for building packages.  
   Installation: `sudo apt-get install -y build-essential`

7. **vcgencmd**: Comes pre-installed on Raspberry Pi OS. Used for fetching CPU temperature.

8. **Speech Recognition Libraries**: Required for `speech_recognition`.  
   Installation: `sudo apt-get install -y libasound2-dev`

9. **I2C Support**: Required for `adafruit_ssd1306` (OLED display).  
   Enable via `raspi-config` or install packages:  
   ```bash
   sudo apt-get install -y i2c-tools
   sudo apt-get install -y python3-smbus
   ```

10. **eSpeak Library**: Required for text-to-speech (`pyttsx3`).  
   Installation: `sudo apt-get install -y libespeak1`

11. **JACK Audio Connection Kit**: Required for handling audio.  
   Installation: `sudo apt-get install -y jackd2`  
   Select `Yes` when prompted to enable realtime privileges.

12. **FLAC Libraries**: Required for handling FLAC audio formats.  
   Installation: `sudo apt-get install -y flac libflac12:armhf`

13. **Git**: Required for cloning the repository.  
    Installation: `sudo apt-get install -y git`

14. **Node.js and npm**: Required for the web interface.  
    Installation: [Follow NodeSource Installation Guide](https://github.com/nodesource/distributions#installation-instructions)

15. **NGINX**: Required for reverse proxy for the web interface.
    Installation: `sudo apt-get install -y nginx`

16. **Rust and Cargo**: Required for installing `spotifyd`.  
    Installation: [Follow Rust Installation Guide](https://www.rust-lang.org/tools/install)

17. **Spotifyd**: Required for Spotify Connect.

18. **Virtual Environment**: Recommended for Python package management.  
   Installation: `sudo apt-get install -y python3-venv`

---

## 🐚 Example setup script:
This script will install all the dependencies and completely set up the project for you. The first time you run it, it will take a while to install all the dependencies. After that, it will be much faster and you can just run it to reinstall the project if you make any changes to the code or want the latest version of the project.

You will need to initialize an environment variable with your OpenAI API Key.  

- *Note: Executing `export` directly in the terminal does not persist after reboot.*  
```bash
export OPENAI_API_KEY="your_openai_api_key_here"
```

Alternatively, you set up the variable in .bashrc file. (recommended)  
- *Put this at the end of your `~/.bashrc` file*
```bash
# export your OpenAI API Key in here to initialize it at boot
export OPENAI_API_KEY="your_openai_api_key_here"

# Optional: Add these aliases to your .bashrc file for easier management
alias gpt-start="sudo systemctl start gpt-home"
alias gpt-restart="sudo systemctl restart gpt-home"
alias gpt-stop="sudo systemctl stop gpt-home"
alias gpt-disable="sudo systemctl disable gpt-home"
alias gpt-status="sudo systemctl status gpt-home"
alias gpt-enable="sudo systemctl enable gpt-home"
alias gpt-log="tail -n 100 -f /home/$(whoami)/gpt-home/events.log"

alias web-start="sudo systemctl start gpt-web"
alias web-restart="sudo systemctl restart gpt-web && sudo systemctl restart nginx"
alias web-stop="sudo systemctl stop gpt-web"
alias web-disable="sudo systemctl disable gpt-web"
alias web-status="sudo systemctl status gpt-web"
alias web-enable="sudo systemctl enable gpt-web"
alias web-log="tail -n 100 -f /var/log/nginx/access.log"
alias web-error="tail -n 100 -f /var/log/nginx/error.log"

alias spotifyd-start="sudo systemctl start spotifyd"
alias spotifyd-restart="sudo systemctl restart spotifyd"
alias spotifyd-stop="sudo systemctl stop spotifyd"
alias spotifyd-disable="sudo systemctl disable spotifyd"
alias spotifyd-status="sudo systemctl status spotifyd"
alias spotifyd-enable="sudo systemctl enable spotifyd"
alias spotifyd-log="journalctl -eu spotifyd"
```
Run `source ~/.bashrc` to apply the changes to your current terminal session.

## setup.sh
Create a script in your home folder with `vim ~/setup.sh` and paste in the following:

<details><summary>👈 View script</summary>
<p>

```bash
#!/bin/bash

# Function to check if service is running and stop it
check_and_stop_service() {
    service=$1

    if sudo systemctl is-active --quiet $service; then
        echo "Stopping $service..."
        sudo systemctl stop $service
    else
        echo "$service is not running."
    fi
}

# Function to check and install a package if it's not installed
check_and_install() {
    package=$1
    install_cmd=$2

    if ! dpkg -l | grep -q $package; then
        echo "Installing $package..."
        eval $install_cmd
    else
        echo "$package is already installed."
    fi
}

# Function to update the system time
update_system_time() {
    echo "Updating system time..."
    check_and_install "ntpdate" "sudo apt-get install -y ntpdate"
    sudo ntpdate -u ntp.ubuntu.com
}

# Check if service is running and stop it
check_and_stop_service "spotifyd"
check_and_stop_service "gpt-home"
check_and_stop_service "gpt-web"

# Set permissions
sudo chown -R $(whoami):$(whoami) $HOME
sudo chmod -R 755 $HOME

# Remove existing local repo if it exists
[ -d "gpt-home" ] && rm -rf gpt-home

# Clone gpt-home repo and navigate into its directory
git clone https://github.com/judahpaul16/gpt-home.git
cd gpt-home

# Update system time
update_system_time

# Update package list
yes | sudo add-apt-repository universe
sudo apt-get update

# Check and install missing dependencies
# Ensure python3.11
if ! command -v pyenv >/dev/null 2>&1 || ! dpkg -l | grep -q "python3.11"; then
    cd ~
    sudo rm -rf ~/.pyenv
    curl https://pyenv.run | bash
    echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
    echo '[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
    echo 'eval "$(pyenv init -)"' >> ~/.bashrc
    source ~/.bashrc
    pyenv install 3.11
    pyenv global 3.11
    pyenv shell 3.11
    pyenv rehash
    cd gpt-home
    echo "python3.11 is installed."
else
    echo "python3.11 is already installed."
fi
check_and_install "python3-venv" "sudo apt-get install -y python3-venv"
check_and_install "python3-dev" "sudo apt-get install -y python3-dev"
check_and_install "portaudio19-dev" "sudo apt-get install -y portaudio19-dev"
check_and_install "alsa-utils" "sudo apt-get install -y alsa-utils"
check_and_install "libjpeg-dev" "sudo apt-get install -y libjpeg-dev"
check_and_install "build-essential" "sudo apt-get install -y build-essential"
check_and_install "libasound2-dev" "sudo apt-get install -y libasound2-dev"
check_and_install "i2c-tools" "sudo apt-get install -y i2c-tools"
check_and_install "python3-smbus" "sudo apt-get install -y python3-smbus"
check_and_install "jackd2" "sudo apt-get install -y jackd2"
check_and_install "libogg0" "sudo apt-get install -y libogg0"
check_and_install "libflac12:armhf" "sudo dpkg -i contrib/libflac12_armhf.deb && sudo apt-get -f install -y && sudo apt-get install -y flac"
check_and_install "flac" "sudo apt-get install -y flac"
check_and_install "libespeak1" "sudo apt-get install -y libespeak1"
check_and_install "cmake" "sudo apt-get install -y cmake"
check_and_install "openssl" "sudo apt-get install -y openssl"
check_and_install "git" "sudo apt-get install -y git"
check_and_install "nginx" "sudo apt-get install -y nginx"
check_and_install "expect" "sudo apt-get install -y expect"
check_and_install "avahi-daemon" "sudo apt-get install -y avahi-daemon avahi-utils"
check_and_install "nodejs" "curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash - && sudo apt-get install -y nodejs"

# Install cargo and rust
if ! command -v cargo &> /dev/null; then
    echo "Installing cargo and rust..."
    curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y

    # Source the environment for cargo and rust
    if [ -f "$HOME/.cargo/env" ]; then
        source $HOME/.cargo/env
    else
        echo "Error: Unable to source Rust environment. Installation may have failed or path is incorrect."
    fi
else
    echo "cargo is already installed."
fi

# Ensure directory exists for the configuration
mkdir -p $HOME/.config/spotifyd

# Install spotifyd using Rust's Cargo
if ! command -v spotifyd &> /dev/null; then
    echo "Installing spotifyd..."
    cargo install spotifyd
    sudo mv $HOME/.cargo/bin/spotifyd /usr/local/bin/
else
    echo "spotifyd is already installed."
fi

# Create Spotifyd configuration (this is just a basic config; adjust accordingly)
cat <<EOF > $HOME/.config/spotifyd/spotifyd.conf
[global]
backend = "alsa" # Or pulseaudio if you use it
device_name = "GPT Home" # Name your device shows in Spotify Connect
bitrate = 320 # Choose bitrate from 96/160/320 kbps
cache_path = "/home/$(whoami)/.spotifyd/cache"
discovery = false
EOF

# Function to setup a systemd service
setup_service() {
    # Parameters
    local SERVICE_NAME=$1
    local EXEC_START=$2
    local DEPENDS=$3
    local ENV=$4
    local HOSTNAME=$5
    local TYPE=$6
    local LMEMLOCK=$7
    local RESTART=$8

    # Stop the service if it's already running
    sudo systemctl stop "$SERVICE_NAME" &>/dev/null

    echo "Creating and enabling $SERVICE_NAME..."
    # Create systemd service file
    cat <<EOF | sudo tee "/etc/systemd/system/$SERVICE_NAME" >/dev/null
[Unit]
Description=$SERVICE_NAME
$DEPENDS
StartLimitIntervalSec=10
StartLimitBurst=10

[Service]
User=$(whoami)
WorkingDirectory=/home/$(whoami)/gpt-home
$EXEC_START
$ENV
$HOSTNAME
$RESTART
$TYPE
$LMEMLOCK

[Install]
WantedBy=multi-user.target
EOF

    # Reload systemd to recognize the new service, then enable and restart it
    sudo systemctl daemon-reload
    sudo systemctl enable "$SERVICE_NAME"
    sudo systemctl restart "$SERVICE_NAME"

    # Wait for 5 seconds and then show the service status
    echo ""
    sleep 5
    sudo systemctl status "$SERVICE_NAME" --no-pager
    echo ""
}

# Setup UFW Firewall
sudo ufw allow ssh
sudo ufw allow 80,443/tcp
sudo ufw allow 5353/udp
echo "y" | sudo ufw enable

# Setup NGINX for reverse proxy
echo "Setting up NGINX..."
sudo tee /etc/nginx/sites-available/gpt-home <<EOF
server {
    listen 80;

    location / {
        proxy_pass http://localhost:8000/;
        proxy_set_header Host \$host;
        proxy_set_header X-Real-IP \$remote_addr;
        proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
    }
}
EOF

# Remove existing symlink if it exists
[ -L "/etc/nginx/sites-enabled/gpt-home" ] && sudo unlink /etc/nginx/sites-enabled/gpt-home

# Symlink the site configuration
sudo ln -s /etc/nginx/sites-available/gpt-home /etc/nginx/sites-enabled

# Test the NGINX configuration
sudo nginx -t

# Remove the default site if it exists
[ -L "/etc/nginx/sites-enabled/default" ] && sudo unlink /etc/nginx/sites-enabled/default

# Reload NGINX to apply changes
sudo systemctl reload nginx

## Setup main app
# Create and activate a virtual environment, then install dependencies
python3 -m venv env
source env/bin/activate
pip install --upgrade pip setuptools
pip install --use-pep517 -r requirements.txt

## Setup Web Interface
# Navigate to gpt-web and install dependencies
cd gpt-web
npm install

# Configure Avahi for gpt-home.local
sudo sed -i 's/#host-name=.*$/host-name=gpt-home/g' /etc/avahi/avahi-daemon.conf
sudo systemctl restart avahi-daemon

## Setup Services
# Setup spotifyd service
setup_service \
    "spotifyd.service" \
    "ExecStart=/usr/local/bin/spotifyd --no-daemon" \
    "Wants=sound.target
    After=sound.target
    Wants=network-online.target
    After=network-online.target" \
    "" \
    "" \
    "" \
    "" \
    "Restart=always
    RestartSec=12"

# Setup gpt-home service
setup_service \
    "gpt-home.service" \
    "ExecStart=/bin/bash -c 'source /home/$(whoami)/gpt-home/env/bin/activate && python /home/$(whoami)/gpt-home/app.py'" \
    "" \
    "Environment=\"OPENAI_API_KEY=$OPENAI_API_KEY\"" \
    "Environment=\"HOSTNAME=$HOSTNAME\"" \
    "Type=simple" \
    "LimitMEMLOCK=infinity" \
    "Restart=always"

# Setup FastAPI service for web interface backend
setup_service \
    "gpt-web.service" \
    "ExecStart=/bin/bash -c 'source /home/$(whoami)/gpt-home/env/bin/activate && uvicorn gpt-web.backend:app --host 0.0.0.0 --port 8000'" \
    "" \
    "Environment=\"OPENAI_API_KEY=$OPENAI_API_KEY\"" \
    "Environment=\"HOSTNAME=$HOSTNAME\"" \
    "Type=simple" \
    "" \
    "Restart=always"

# Mask systemd-networkd-wait-online.service to prevent boot delays
sudo systemctl mask systemd-networkd-wait-online.service
```

</p>
</details>

Be sure to make the script executable to run it
```bash
cd ~
chmod +x setup.sh
./setup.sh
```

---

## ✅ Compatibility

<p align="center">
    <table>
    <tr valign="top">
        <!-- Raspberry Pi -->
        <td>
        <table>
            <tr><th colspan="2">Raspberry Pi</th></tr>
            <tr><td>3B</td><td>✅</td></tr>
            <tr><td>3B+</td><td>✅</td></tr>
            <tr><td>4B</td><td>✅</td></tr>
            <tr><td>Zero 2 W</td><td>❔</td></tr>
            <tr><td>Orange Pi 3B</td><td>✅</td></tr>
        </table>
        </td>
        <!-- Python -->
        <td>
        <table>
            <tr><th colspan="2">Python</th></tr>
            <tr><td>3.7</td><td>❌</td></tr>
            <tr><td>3.8</td><td>✅</td></tr>
            <tr><td>3.9</td><td>✅</td></tr>
            <tr><td>3.10</td><td>✅</td></tr>
            <tr><td>3.11</td><td>✅</td></tr>
            <tr><td>3.12</td><td>❌</td></tr>
        </table>
        </td>
        <!-- Operating Systems -->
        <td>
        <table>
            <tr><th colspan="2">Operating System</th></tr>
            <tr><td>Ubuntu 22.04</td><td>❔</td></tr>
            <tr><td>Ubuntu 23.04</td><td>✅</td></tr>
            <tr><td>Ubuntu 24.04</td><td>❔</td></tr>
            <tr><td>Raspbian Buster</td><td>❔</td></tr>
            <tr><td>Raspbian Bullseye</td><td>❔</td></tr>
            <tr><td>CentOS 7</td><td>❔</td></tr>
            <tr><td>CentOS 8</td><td>❔</td></tr>
            <tr><td>CentOS 9</td><td>❔</td></tr>
        </table>
        </td>
        <!-- Node.js -->
        <td>
        <table>
            <tr><th colspan="2">Node.js</th></tr>
            <tr><td>17.x</td><td>❔</td></tr>
            <tr><td>18.x</td><td>✅</td></tr>
            <tr><td>19.x</td><td>❔</td></tr>
            <tr><td>20.x</td><td>❔</td></tr>
            <tr><td>21.x</td><td>❔</td></tr>
            <tr><td>22.x</td><td>❔</td></tr>
        </table>
        </td>
    </tr>
    </table>
</p>

---

## 📚 Useful Documentation

<table>
<tr>
<td>

- [OpenAI API Docs](https://platform.openai.com/docs/introduction)
- [Raspberry Pi Docs](https://www.raspberrypi.com/documentation)
- [Node.js Docs](https://nodejs.org/en/docs/)
- [npm Docs](https://docs.npmjs.com/)
- [NGINX Docs](https://nginx.org/en/docs/)
- [React Docs](https://reactjs.org/docs/getting-started.html)
- [FastAPI Docs](https://fastapi.tiangolo.com/)
- [Ubuntu Server Docs](https://ubuntu.com/server/docs)

</td>
<td>

- [GPIO Pinout](https://www.raspberrypi.com/documentation/computers/images/GPIO-Pinout-Diagram-2.png)
- [Adafruit SSD1306 Docs](https://circuitpython.readthedocs.io/projects/ssd1306/en/latest/)
- [pyttsx3 Docs](https://pypi.org/project/pyttsx3/)
- [I2C Docs](https://i2c.readthedocs.io/en/latest/)
- [ALSA Docs](https://www.alsa-project.org/wiki/Documentation)
- [PortAudio Docs](http://www.portaudio.com/docs/v19-doxydocs/index.html)
- [SpeechRecognition Docs](https://pypi.org/project/SpeechRecognition/)

</td>
<td>

- [Spotify API Docs](https://developer.spotify.com/documentation/web-api/)
- [Spotify API Python Docs (Spotipy)](https://spotipy.readthedocs.io/)
- [Spotifyd Docs](https://docs.spotifyd.rs/)
- [Phillips Hue API Docs](https://developers.meethue.com/develop/get-started-2/)
- [Phillips Hue Python API Docs](https://github.com/studioimaginaire/phue)
- [OpenWeatherMap API Docs](https://openweathermap.org/api/one-call-3)
- [Open-Meteo API Docs](https://open-meteo.com/en/docs)
- [Fritzing Schematics](https://fritzing.org/)

</td>
</tr>
</table>

---

## 🤝 Contributing
Contributions are certainly welcome! Please read the [contributing guidelines](CONTRIBUTING.md) for more information on how to contribute.

## 📜 License
This project is licensed under the GNU GPL v3.0 License - see the [LICENSE](LICENSE) file for details.
