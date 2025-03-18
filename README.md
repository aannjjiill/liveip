# Live IP Checker (`liveip.py`)

A Python script to check if IPs from a file or an IP range (CIDR) are active by sending a ping request.

---

## Features
- Reads IPs from a file.
- Checks all IPs in a given CIDR range (e.g., `1.2.3.0/24`).
- Uses ping to determine if the IPs are live.
- Displays active and inactive IPs.

---

## Installation

1. **Clone the repository**:
   ```bash
   git clone https://github.com/aannjjiill/liveip.git
   cd liveip
   ```

2. **Ensure the script has executable permissions**:
   ```bash
   chmod +x liveip.py
   ```

3. **Run the script with the required arguments**:
   - To check IPs from a file:
     ```bash
     ./liveip.py -f ips.txt
     ```

   - To check IPs from a CIDR range:
     ```bash
     ./liveip.py -r 192.168.1.0/24
     ```

---

## Usage

### Command Line Arguments

- `-f`, `--file`: Path to the file containing the list of IP addresses.
- `-r`, `--range`: CIDR notation for an IP range (e.g., `192.168.1.0/24`).

### Examples

- Check IPs from a file:
  ```bash
  ./liveip.py -f ips.txt
  ```

- Check IPs from a CIDR range:
  ```bash
  ./liveip.py -r 192.168.1.0/24
  ```

---

## Script

```python name=liveip.py
#!/usr/bin/env python3

import argparse
import subprocess
import os
import ipaddress

def ping_ip(ip):
    """Ping a given IP address and return True if it's live."""
    try:
        output = subprocess.check_output(['ping', '-c', '1', ip], stderr=subprocess.STDOUT, universal_newlines=True)
        if "1 packets transmitted, 1 received" in output:
            return True
    except subprocess.CalledProcessError:
        pass
    return False

def check_ips(ips):
    """Check a list of IPs and print their status."""
    active_ips = []
    for ip in ips:
        if ping_ip(ip):
            print(f"{ip} is active")
            active_ips.append(ip)
        else:
            print(f"{ip} is inactive")
    return active_ips

def check_ips_from_file(file_path):
    """Read IPs from a file and check their status."""
    try:
        with open(file_path, 'r') as file:
            ips = [line.strip() for line in file if line.strip()]
        return check_ips(ips)
    except FileNotFoundError:
        print(f"Error: The file {file_path} does not exist.")
        return []
    except Exception as e:
        print(f"An error occurred: {e}")
        return []

def check_ips_from_range(ip_range):
    """Generate IPs from a given CIDR range and check their status."""
    try:
        network = ipaddress.ip_network(ip_range, strict=False)
        ips = [str(ip) for ip in network.hosts()]
        return check_ips(ips)
    except ValueError:
        print("Error: Invalid CIDR notation provided.")
        return []

def main():
    parser = argparse.ArgumentParser(
        description='Check the availability of IP addresses from a file or a specified IP range.',
        epilog='Example usage:\n  python3 liveip.py -f ips.txt\n  python3 liveip.py -r 192.168.1.0/24',
        formatter_class=argparse.RawTextHelpFormatter
    )
    
    parser.add_argument('-f', '--file', type=str, help='Path to the file containing the list of IP addresses.')
    parser.add_argument('-r', '--range', type=str, help='CIDR notation for an IP range (e.g., 192.168.1.0/24).')

    args = parser.parse_args()

    if args.file and args.range:
        print("Error: Please provide only one input method (either a file or an IP range).")
        exit(1)
    elif args.file:
        active_ips = check_ips_from_file(args.file)
    elif args.range:
        active_ips = check_ips_from_range(args.range)
    else:
        print("Error: You must specify either a file (-f) or an IP range (-r). Use -h for help.")
        exit(1)

    print(f"\n✅ Active IPs: {active_ips}")

if __name__ == '__main__':
    main()
```
