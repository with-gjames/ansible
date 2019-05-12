# Ansible Code and Cookbooks

In learning Ansible I have come across a few problems that are not well documented.  Questions like, How can I know all the interfaces on a Cisco device programtically?   Or, let's say you wanted to take the output of SHOW INTERFACE STATUS | IN TRUNK and use the interface names to make a change to all Trunk ports. How can I parse those interface names so Ansible can use them in a task.   

## Basic Cisco Report
  This playbook will use ios_facts to generate an interface report from a Cisco Router or Switch.  Reports are output into markdown (md) and CSV.   The reports can easily be modified with any data pulled from ios_facts based on user need.   

  We use .iteritems() to iterate through each interface in the ansible_net_interfaces dictionary.  {{ key }} is the name of the interface.  

     {% for key,value in ansible_net_interfaces.iteritems() %}
     {{  key }},{{ value.macaddress }},{{ value.description }}
     {{ endfor }}

md reports can be displayed in a wiki or other wegpage to aid in troubleshooting.  CSV files can be used in auditing or in conjunction with a config generator.   

## Parse SHOW command for Interface names
 
  It is often necessary to update an unknown interfaces on a Cisco switch or router with a new command based on some criteria, such as "all trunk ports" or "all interfaces with 'WAN' in the description".   You could put all the interface names into global vars and iterate through them, but that is time consuming and prone to error.   It would be better if we could take the output of a SHOW command that displays our target interface names, and then iterate through those.  

  For example, given the output of show interface status | include ^[Gi|Fa].*trunk

~~~~
Switch#show interfaces status | include ^[Gi|Fa].*trunk
Gi1/2                      notconnect   trunk            auto   auto No Gbic
Gi5/1                      notconnect   trunk            auto   auto 10/100/1000-TX
Gi5/2                      notconnect   trunk            auto   auto 10/100/1000-TX
~~~~
  
   In Ansible we register that output into a variable called trunk_ports

~~~~
- name:  LIST TRUNK PORTS
    ios_command:
    commands:
        - show interface status | include ^[Gi|Fa].*trunk
    register: trunk_ports          
~~~~



We can parse the output with split() 
"{{ trunk_ports.stdout_lines[0] }} will convert the output into lines.  Each line is procssed individually.
Split will parse the line into tokens.  We want the first token, so we referenece it with split()[0]  
And because we want to remain idempotent, we convert the Gi or Fa into the full interface name with a regex


~~~~
- name: ADD AUTO QOS TO TRUNK PORTS
ios_config:
    lines:
    - auto qos trust dscp
    parents: interface {{ item.split()[0] | 
            regex_replace('^Gi','GigabitEthernet') |
            regex_replace('^Fa','FastEthernet') }}
            # Full interface names must be used for idempotency
            # Add more regex lines for your other interface types
with_items: "{{ trunk_ports.stdout_lines[0] }}" `
~~~~   

You will have to get creative with your show commands to get the output formatted into the table form above, but as long as you can do that, the output can parsed in any way.

## Authors

* **George James**


## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details

