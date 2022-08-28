# Port2Excel
This python code scans Cisco switches and gather information about your ports into an Excel file



==== Requirements ===
The code requires Python 3.10 interpreter + the following libraries:
  1. netmiko --> to establish SSH sessions, which by itself uses the following libraries:
    1.1 paramkio --> the original ssh connection handler
    1.2 textFSM --> it is a google project to parse regular semi-structured outputs into python lists and dictionaries
    1.3 ntc_templates --> it is a project by networktocode.com to create textFSM tamplates for major network vendors (cisco, aruba, palo alto...etc.)
  2. Re --> REGEX to match strings
  3. openpyxl --> to load and create Excel files
  4. mac_vendor_lookup --> to find the manufacturer using the reserved OUI in the MAC address



=== Getting Started ===
Pleae ensure you fill the Inventory.xlsx file with your current ineventory:
  1. ASW --> Access Layer Switch you are getting port information from
  2. DSW --> Distribution Layer Switch to get ARP table for MAC-to-IP resolution

Pleae fill the username/password in the connection parameters. These credentails are not best stored within the code, however, in the future iterations we will ensure they are securly inputted during the execution of the code.

Please use the ntc_template templates files given in the repository since the default templates in the library will not work with this code.

