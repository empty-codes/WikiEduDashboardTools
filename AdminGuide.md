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
        - [Some useful commands](#some-useful-commands)
   - [Cloud VPS](#cloud-vps)
3. [Troubleshooting](#troubleshooting)
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
- View Kubernetes namespace status:  [Tool Namespace Status](https://k8s-status.toolforge.org/namespaces/tool-wikiedudashboard/)
- View pod details:  [Tool Pod Details](https://k8s-status.toolforge.org/namespaces/tool-wikiedudashboard/pods/)

#### Some useful Commands
- Check recent logs: `toolforge webservice logs -l 100`
- Follow logs in real time: `toolforge webservice logs -f`
- List active jobs: `toolforge jobs list`
- To restart cronjobs / continuous jobs: `toolforge jobs restart <jobname>`
- To view internal job logs: `toolforge jobs logs`
- To view job quotas: `toolforge jobs quota`

### Cloud VPS

## Troubleshooting


## More Resources
- [Toolforge Documentation](https://wikitech.wikimedia.org/wiki/Help:Toolforge)
- [Cloud VPS Documentation](https://wikitech.wikimedia.org/wiki/Help:Cloud_VPS)
