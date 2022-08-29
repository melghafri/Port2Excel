# Port2Excel
This python code automates the process of collecting switch ports information and exports all outputs into an Excel sheet.


The code requires Python 3.10 virtual environment (venv) interpreter + the following libraries:
  1. netmiko https://github.com/ktbyers/netmiko --> to establish SSH sessions to popular networking vendors, uses paramiko in the backend
  2. paramkio https://github.com/paramiko/paramiko --> the original ssh connection handler
  3. textfsm https://github.com/google/textfsm --> a google project to parse regular semi-structured outputs into python lists and dictionaries
  4. ntc_templates https://github.com/networktocode/ntc-templates --> provides ready-made textFSM templates for major network vendors (cisco, aruba, palo alto...etc.)
  5. re https://github.com/AstunTechnology/python-basics --> to match or search in strings
  6. openpyxl https://github.com/theorchard/openpyxl --> to load and create Excel files
  7. mac_vendor_lookup https://github.com/bauerj/mac_vendor_lookup --> to find the manufacturer using the reserved OUI in the MAC address

*Details about the lib versions are in Requirements.txt file.

=== Getting Started ===

Pleae ensure you fill the Inventory.xlsx file with your current ineventory. Use the file in the repo as a template. Please ensure you have ASW/DSW in the hostname of the switches as below:
  1. ASW --> Access Layer Switch you are getting port information from
  2. DSW --> Distribution Layer Switch to get ARP table for MAC-to-IP resolution

*in case you don't need IP reolution, you can ignore DSW switches.

Pleae fill the username/password in the connection parameters. These credentails are not best stored within the code, however, in the future iterations we will ensure they are securly inputted during the execution of the code.

Please use the ntc_template templates files given in the repository since the default templates in the library will not work with this code.

You need to put them into the following directory C:\(your python venv directory)\Lib\site-packages\ntc_templates\templates


=== How the code works ==-

The code uses Google's TextFSM library which parses (converts) a the semi-structured show command outputs into a python list of dictionaries based on a template you feed it with.

the TextFSM itself is like the base engine that rruns as a state machine (start, record ..etc) you need to give it the insturction how to record the intersting values using the template. The output is usually in the format of a pythong list of dictionaries

Example how TextFsm works with Cisco switches is below:

Here is a simple show interface status on a Cisco Catalyst switch:


    switch01#show interfaces status

    Port      Name               Status       Vlan       Duplex  Speed Type

    Fa1/0/1                      notconnect   3            auto   auto 10/100BaseTX

    Fa1/0/2                      notconnect   4            auto   auto 10/100BaseTX

    Fa1/0/3                      notconnect   3            auto   auto 10/100BaseTX


The actual template used by TextFSM to parse the output is  below:

    Value PORT (\S+)
    Value NAME (.+?)
    Value STATUS (err-disabled|disabled|connected|notconnect|inactive|up|down|monitoring)
    Value VLAN (\S+)
    Value DUPLEX (\S+)
    Value SPEED (\S+)
    Value TYPE (.*)
    Value FC_MODE (\S+)

    Start
      ^Load\s+for\s+
      # Capture time-stamp if vty line has command time-stamping turned on
      ^Time\s+source\s+is
      ^-+\s*$$
      ^Port\s+Name\s+Status\s+Vlan\s+Duplex\s+Speed\s+Type -> Interfaces
      ^\s*$$
      ^. -> Error

    Interfaces
      #Match fc...
      ^\s*${PORT}\s+is\s+${STATUS}\s+Port\s+mode\s+is\s+${FC_MODE}\s*$$ -> Record
      ^\s*${PORT}\s+is\s+${STATUS}\s+\(${TYPE}\)\s*$$ -> Record
      ^\s*${PORT}\s+${STATUS}\s+${VLAN}\s+${DUPLEX}\s+${SPEED}\s*${TYPE}$$ -> Record
      ^\s*${PORT}\s+${NAME}\s+${STATUS}\s+${VLAN}\s+${DUPLEX}\s+${SPEED}\s*${TYPE}$$ -> Record
      ^-+
      ^\s*$$
      ^. -> Error


The parsed output will be in a python list of dictionaries like below:

    [
    {'port': 'Fa1/0/1 , 'name': '', 'status': 'notconnect', 'vlan': '3','duplex': 'auto','speed: 'auto','type: '10/100BaseTX'},
    {'port': 'Fa1/0/2 , 'name': '', 'status': 'notconnect', 'vlan': '4','duplex': 'auto','speed: 'auto','type: '10/100BaseTX'},
    {'port': 'Fa1/0/3 , 'name': '', 'status': 'notconnect', 'vlan': '3','duplex': 'auto','speed: 'auto','type: '10/100BaseTX'},
    ]


In the code, you will see 2 main functions:
 1. def search_for_someones_attr(list_of_dict, key_a, value_a, key_b):
 2. def search_for_someones_multi_attr(list_of_dict, key_a, value_a, key_b):

both functions do almost the same thing, which is taking a list of dictionaries (list_of_dict) that we got from TextFSM and search for an individual dictionary and then retuns a value in that particular dictionary.

We use these functions to regularly to gather information about a spefic port (e.g. Gi1/0/15) to get its learned MAC or allocated PoE power for example.

the code itself writes all the output in an excel sheet, where each row represent 1 switch port. The columns are the individual parameters. Currently the code captures the following switch port parameters:

    Column C: port number
    Column D: port description
    Column E: port status
    Column F: switch port mode
    Column G: VLAN(s) --> since port can be a trunk
    Column H: VLAN name(s)
    Column I: Voice VLAN
    Column J: Voice VLAN name
    Column K: MAC(s) learned
    Column L: Manufacturer(s)
    Column M: IP(s) resolved
    Column N: CDP neighbor name(s)
    Column O: CDP neighbor model(s)
    Column P: CDP neighbor management IP(s)
    Column Q: Last input(s)
    Column R: Last output(s)
    Column S: Power admin status
