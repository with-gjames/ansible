# Ansible Code and Cookbooks



## Basic Cisco Report
  This playbook will use ios_facts to generate an interface report from a Cisco Router or Switch.  Reports are output into markdown (md) and CSV.   The reports can easily be modified with any data pulled from ios_facts based on user need.   

  We use .iteritems() to iterate through each interface in the ansible_net_interfaces dictionary.  {{ key }} is the name of the interface.  

    {% for key,value in ansible_net_interfaces.iteritems() %}
     {{  key }},{{ value.macaddress }},{{ value.description }}
     {{ endfor }}

md reports can be displayed in a wiki or other wegpage to aid in troubleshooting.  CSV files can be used in auditing or in conjunction with a config generator.   




## Authors

* **George James** - *Initial work* 


## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details

