# telnet-check-mac-address
This code allows you to view connected mac-addresses for a certain time, 
which is indicated in the INTERVAL_TIME line
after this time, the code writes whether the deletion or addition 
of mac-address was noticed during the time that you 
specified



###############################################################################################################################################




import telnetlib
import time
import re

MAC_DATA = []
INTERVAL_TIME = time
HOST = 'host'

def format_mac_address(mac_address):
    return '-'.join([mac_address[i:i+2] for i in range(0, len(mac_address), 3)])

def telnet_and_get_mac_table(host, port, command):
    """Establish Telnet connection and retrieve MAC addresses."""
    try:
        tn = telnetlib.Telnet(host, port, timeout=5)
        tn.read_until(b"login:", timeout=2).decode('ascii')
        tn.write(b"admin\n")
        tn.read_until(b"Password:", timeout=2).decode('ascii')
        tn.write(b"admin\n")
        tn.write(b"show mac-address-table\n")
        tn.read_until(b"name", timeout=2).decode('ascii')
        tn.write(command)
        mac_table = tn.read_until(b"name", timeout=2).decode('ascii')
        tn.close()
        mac_data = []
        splited = mac_table.split(' ')
        for i in splited:
            mac_format = r"[0-9A-Fa-f]{2}(?:[:-][0-9A-Fa-f]{2}){5}"
            if re.match(mac_format, i):
                mac_data.append(i)
        return mac_data
    except Exception as e:
        print('Failed to connect to the device:', e)
        return None

def check_table(old_mac_data, new_mac_data):
    """Compare MAC address arrays and print changes."""
    if old_mac_data != new_mac_data:
        print(f"MAC addresses have changed: Old MACs:{old_mac_data}\nNew MACs:{new_mac_data}")
    added_macs = list(set(new_mac_data) - set(old_mac_data))
    removed_macs = list(set(old_mac_data) - set(new_mac_data))
    if added_macs:
        print(f"The following MAC addresses have been added: {', '.join(added_macs)}")
    if removed_macs:
        print(f"The following MAC addresses have been removed: {', '.join(removed_macs)}")
    else:
        print("MAC addresses remain the same.")

def main():
    while True:
     command = b"show mac-address-table\n"
     old_mac_data = telnet_and_get_mac_table(HOST, 23, command)
     time.sleep(INTERVAL_TIME)
     new_mac_data = telnet_and_get_mac_table(HOST, 23, command)
     if old_mac_data is not None and new_mac_data is not None:
        check_table(old_mac_data, new_mac_data)
main()# mac-address-check
