# managepi
A open-source local network dashboard for managing your Linux devices.

# Client: Installing dependencies

```
sudo apt upgrade
sudo apt install python3-pip
pip3 install flask psutil
mkdir ManagePi
cd ManagePi
```

Add the following code to the ``ManagePi`` folder, name the script ``client_app.py``

```python
from flask import Flask, jsonify
import os
import psutil
import subprocess

app = Flask(__name__)

@app.route('/cpu-usage', methods=['GET'])
def get_cpu_usage():
    cpu_usage = psutil.cpu_percent(interval=1)
    return jsonify({'cpu_usage': cpu_usage})

@app.route('/memory', methods=['GET'])
def get_memory():
    disk_usage = psutil.disk_usage('/')
    return jsonify({
        'total_memory': disk_usage.total,
        'used_memory': disk_usage.used,
        'free_memory': disk_usage.free,
        'percent_used': disk_usage.percent
    })

@app.route('/cpu-temperature', methods=['GET'])
def get_cpu_temperature():
    try:
        with open('/sys/class/thermal/thermal_zone0/temp', 'r') as file:
            temp_c = int(file.read()) / 1000.0
    except FileNotFoundError:
        temp_c = None
    return jsonify({'cpu_temperature': temp_c})

@app.route('/restart', methods=['POST'])
def restart():
    subprocess.run(["sudo", "reboot"])
    return jsonify({'status': 'restarting'}), 202

@app.route('/shutdown', methods=['POST'])
def shutdown():
    subprocess.run(["sudo", "shutdown", "now"])
    return jsonify({'status': 'shutting down'}), 202

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

# Client: Making the code a background service

```
sudo nano /etc/systemd/system/client_app.service
```

Add the following code into the file and change the paths

```
[Unit]
Description=Client App Service
After=network.target

[Service]
ExecStart=/usr/bin/python3 /path/to/client_app.py
Restart=always

[Install]
WantedBy=multi-user.target

```

Enable the service by doing the following commands

```
sudo systemctl enable client_app.service
sudo systemctl start client_app.service
```

Test if the webserver is working by using ``curl`` on another computer connected to the same network

```
curl http://your-pi-ip-address/cpu-usage
```

You should get an output like this

```
{"cpu_usage": 25.3}
```

# Dashboard
