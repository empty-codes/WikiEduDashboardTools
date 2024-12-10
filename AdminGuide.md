# Admin Guide for WikiEduDashboardTools

This guide provides detailed instructions for setting up, managing, and troubleshooting WikiEduDashboardTools on Toolforge and Cloud VPS.

## Table of Contents
1. [Setup Instructions](#setup-instructions)
   - [Toolforge Setup](#toolforge-setup)
   - [Cloud VPS Setup](#cloud-vps-setup)
2. [Managing the System](#managing-the-system)
   - [Toolforge](#toolforge)
        - [Starting, Stopping, and Restarting Services](#starting-stopping-and-restarting-services)
        - [Monitoring and Logs](#monitoring-and-logs)
        - [Some useful toolforge commands](#some-useful-toolforge-commands)
   - [Cloud VPS](#cloud-vps)
        - [Starting, Stopping, and Restarting Instances](#starting-stopping-and-restarting-instances)
        - [Monitoring](#monitoring)
        - [Some useful commands](#some-useful-commands)
3. [Troubleshooting](#troubleshooting)
    - [Web Server Issues](#web-server-issues)
    - [Database Issues](#database-issues)
    - [Data Dumps and Recovery](#data-dumps-and-recovery)
4. [More Resources](#more-resources)


## Setup Instructions

### Toolforge Setup
1. Clone the repository in the tool's home directory:
   ```bash
   git clone https://github.com/WikiEducationFoundation/WikiEduDashboardTools.git
   ```
2. Start the web server:
   ```bash
   webservice start
   ```

### Cloud VPS Setup
1. Create a Debian server with the `web` security group.
2. Add DNS for an external URL (like dashboard-replica-endpoint.wmcloud.org)
3. Install required packages:
   ```bash
   sudo apt install apache2 php libapache2-mod-php php-mysql
   ```
4. Clone the repository into `/var/www/`:
   ```bash
   git clone https://github.com/WikiEducationFoundation/WikiEduDashboardTools.git /var/www/WikiEduDashboardTools
   ```
4. Configure the web server:

   - Disable the default site and add a new site configuration file (`dashboard-tools.conf`):
    ``` 
        <VirtualHost *:80>
            ServerName dashboard-replica-endpoint.wmcloud.org
            ServerAdmin sage@wikiedu.org   
            DocumentRoot /var/www/WikiEduDashboardTools/public_html
            ErrorLog ${APACHE_LOG_DIR}/error.log
            CustomLog ${APACHE_LOG_DIR}/access.log combined
        </VirtualHost> 
    ```
   - Increase the PHP memory_limit in `/etc/php/7.4/apache2/php.ini` or a similar location (to 4096M):
     ```ini
     memory_limit = 4096M
     ```
     - This ensures that queries that return very large amounts of data won't run PHP out of memory while converting query results to JSON.
   - Enable the new site, and visit /index.php

5. Add Replica database credentials:
   - Copy `replica.my.cnf` from a Toolforge tool account into the tool's root directory.
6. Test a query endpoint, for example: 
   - `/revisions.php?lang=en&project=wikipedia&usernames[]=Ragesoss&start=20140101003430&end=20171231003430`


## Managing the System

### Toolforge

#### Starting, Stopping, and Restarting Services
- **Start the service**: `webservice start`
- **Stop the service**: `webservice stop`
- **Restart the service**: `webservice restart`
- **Check server status**: `webservice status`

#### Monitoring and Logs
- [Kubernetes Namespace Details](https://k8s-status.toolforge.org/namespaces/tool-wikiedudashboard/)
- [Kubernetes Pod Details](https://k8s-status.toolforge.org/namespaces/tool-wikiedudashboard/pods/wikiedudashboard-5954f86c86-pm8d5/)

#### Some useful toolforge commands
- Check recent logs: `toolforge webservice logs -l 100`
- Follow logs in real time: `toolforge webservice logs -f`
- List active jobs: `toolforge jobs list`
- To restart cronjobs / continuous jobs: `toolforge jobs restart <jobname>`
- To view internal job logs: `toolforge jobs logs`
- To view job quotas: `toolforge jobs quota`

### Cloud VPS

#### Starting, Stopping, and Restarting Instances
- **Start instance**: `openstack server start <id>`
- **Stop instance**: `webservice stop`
- **Restart instance**: `openstack server reboot <id>`
- **Check status of all instances (ACTIVE, SHUTOFF, PAUSED)**: `openstack server list --host hostname --all-projects`

#### Monitoring
- [Grafana](https://grafana.wmcloud.org/d/0g9N-7pVz/cloud-vps-project-board?orgId=1&var-project=globaleducation)
- [Server Admin Logs (SAL)](https://sal.toolforge.org/globaleducation)
- [Alerts](https://prometheus-alerts.wmcloud.org/?q=%40state%3Dactive&q=project%3Dglobaleducation)
- [Puppet agent logs for the globaleducation project](https://grafana.wmcloud.org/d/SQM7MJZSz/cloud-vps-puppet-agents?orgId=1&var-project=globaleducation&from=now-2d&to=now) 


#### Some useful commands
- Check memory: `free -m`
- Check processor(s): `cat /proc/cpuinfo  | grep ^processor`
- Check status of systemd unit: `systemctl status <unit_name>`
- Check logs of systemd unit: `journalctl -u <unit_name> -n 1000`
- Check all cinder services and logs: `sudo systemctl status cinder* -l`
- Check cinder backup list for available volumes: `openstack volume backup list | grep -iv available`
- List all available flavors of instances: `sudo wmcs-openstack flavor list`
- Check the current status of an instance: `OS_PROJECT_ID=PROJECT-ID sudo wmcs-openstack server show SERVER-ID`
- Resize an instance to a new flavor using ID: `OS_PROJECT_ID=PROJECT-ID sudo wmcs-openstack server resize --flavor FLAVOR-ID SERVER-ID`
- Show messages about Openstack APIs being up or down: `$ sudo tail /var/log/haproxy/haproxy.log`


## Troubleshooting

### Web Server Issues
- **Internal Server Error**: Restart the web server.  
- **Unresponsive Web Service**:  
  - Usually caused by high-activity events or surges in ongoing activity, leading to system overload.  
    - **Solution**: Reboot the VM (instance) running the web server.  
    - The web service typically recovers within a few hours.  

### Database Issues
- **Full Disk**: Free up space by deleting temporary tables.  
- **High-Edit / Long Courses Causing Errors**:  
  - Consider turning off the 'long' and 'very_long_update' queues.   
- **Stuck Transactions**: If results in the Rails server becoming unresponsive, restart MySQL.  
- **Database Errors**:  
  - Verify that the app and database server versions are compatible.  

### Data Dumps and Recovery
- **Performing a Dump for a table**:  
  1. Put the database in `innodb_force_recovery=1` mode. 
        - Note: `OPTIMIZE TABLE revisions;` cannot run in recovery mode because the database is read-only.  
  2. Start the dump process.  
  3. Once the dump is complete, drop the table.  
  4. Remove the database from recovery mode and restore the table.  

### Third-Party Dependencies
Issues could also be caused by maintenance or outages in third-party dependencies such as Openstack, Toolforge, or other services.


## More Resources
- [Toolforge Documentation](https://wikitech.wikimedia.org/wiki/Help:Toolforge)
- [Cloud VPS Documentation](https://wikitech.wikimedia.org/wiki/Help:Cloud_VPS)
- [Cloud VPS Admin Documentation](https://wikitech.wikimedia.org/wiki/Portal:Cloud_VPS/Admin)
- [Details of most recent P&E server update](https://github.com/WikiEducationFoundation/WikiEduDashboard/commit/df271f1c54fd0520e42445fcc88f19b6d03a603b#diff-f8eaa8feeef99c2b098e875ccdace93998b84eeb4110dc9f49b1327df7d96e21)
