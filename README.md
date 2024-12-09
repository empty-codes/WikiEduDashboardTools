# WikiEduDashboardTools
## Introduction
A set of PHP endpoints for pulling revision and article data from the Replica databases in the Wikimedia Cloud environment. [public_html/common.php](https://github.com/WikiEducationFoundation/WikiEduDashboardTools/blob/master/public_html/common.php) is where namespaces get set for the database queries that get made by the replica endpoint.

* Phabricator Project Link: https://phabricator.wikimedia.org/project/manage/1052/
* The Wiki Education Dashboard runs in Cloud VPS in the [globaleducation](https://openstack-browser.toolforge.org/project/globaleducation) project.

* Deployed at: https://wikiedudashboard.toolforge.org/ and https://replica-revision-tools.wmcloud.org/?


## Deployment Guide
Note: You will need a Wikimedia account and a Wikimedia developer account to use Toolforge or the Cloud VM

### Setup on Toolforge
Toolforge is a free cloud hosting platform within Wikimedia Cloud Services (WMCS) tailored for hosting and managing community-developed tools and bots. It simplifies the process of deploying and maintaing tools for Wikimedia contributors.

* Clone the git repo in the tool's home directory
* `webservice start`

* Start the web server
`webservice start`

* Stop the web server
`webservice stop`

* Restart the web server
`webservice restart`

* Check the status of the web server
`webservice status`

* Note: The lighttpd + PHP 7.3 is the default web server and serves content from the [/public_html](https://github.com/WikiEducationFoundation/WikiEduDashboardTools/tree/master/public_html) directory 

* For any general problems, check here for troubleshooting tips: https://wikitech.wikimedia.org/wiki/Help:Toolforge/Troubleshooting

### Setup on Wikimedia Cloud VM
* Create a Debian server with the `web` security group
* Add DNS for an external URL (like dashboard-replica-endpoint.wmcloud.org)
* `sudo apt install apache2 php libapache2-mod-php php-mysql`
* Clone the git repo into /var/www/
* Disable the default site and add a new one (dashboard-too.conf)
```
<VirtualHost *:80>
    ServerName dashboard-replica-endpoint.wmcloud.org
    ServerAdmin sage@wikiedu.org   
    DocumentRoot /var/www/WikiEduDashboardTools/public_html
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
* Increase the PHP memory_limit in `/etc/php/7.4/apache2/php.ini` or similar location (to 4096M)
  * This ensures that queries that return very large amounts of data won't run PHP out of memory while converting query results to JSON.
* Enable the new site, and visit /index.php.
* Add Replica database credentials: copy `replica.my.cnf` from a Toolforge tool account into the tool's root directory.
* Test a query endpoint, like `/revisions.php?lang=en&project=wikipedia&usernames[]=Ragesoss&start=20140101003430&end=20171231003430`


## Debugging Tips
(to be reviewed as I have not confirmed if they actually work)
* To view the status of the `tool-wikiedudashboard` Kubernetes namespace, visit: https://k8s-status.toolforge.org/namespaces/tool-wikiedudashboard/

* To view the `tool-wikiedudashboard` pod, visit: https://k8s-status.toolforge.org/namespaces/tool-wikiedudashboard/pods/wikiedudashboard-5954f86c86-pm8d5/

* To view logged errors and other information run: `toolforge webservice logs` `-f` to follow in real time, `-l [num]` to see `num` of the newest lines 

* To list existing jobs: `toolforge jobs list`, `--output long` for more information, `jobs dump` in yaml format

* To restart cronjobs / continuous jobs: `toolforge jobs restart <jobname>`

* To view internal job logs: `toolforge jobs logs`

* To view job quotas: `toolforge jobs quota`

## Example Queries
* Example revisions.php query:
  https://replica-revision-tools.wmcloud.org/revisions.php?lang=en&project=wikipedia&usernames[]=Ragesoss&start=20140101003430&end=20171231003430
  

* Tip: Naive queries are slower so be more specific?
## Troubleshooting

* More information about the `webservice` command can be found here: https://wikitech.wikimedia.org/wiki/Help:Toolforge/Web

* Using CDNs might result in a 504 Gateway Time-out error when accessing the site; CDNs keep track of traffic which violates the Wikimedia Cloud terms of service

## More information
* https://wikitech.wikimedia.org/wiki/Help:Cloud_Services_introduction#What_is_the_difference_between_Cloud_VPS_and_Toolforge?
* [Commit](https://github.com/WikiEducationFoundation/WikiEduDashboard/commit/df271f1c54fd0520e42445fcc88f19b6d03a603b#diff-f8eaa8feeef99c2b098e875ccdace93998b84eeb4110dc9f49b1327df7d96e21) detailing most recent updates for the P & E server: 
* Wiki Education Foundation Server Configuration: [SERVER_CONFIG.md](https://github.com/WikiEducationFoundation/WikiEduDashboard/blob/master/server_config/SERVER_CONFIG.md),
 the main web server configuration files are: [apache2.conf](https://github.com/WikiEducationFoundation/WikiEduDashboard/blob/master/server_config/apache2.conf) , [passenger.load](https://github.com/WikiEducationFoundation/WikiEduDashboard/blob/master/server_config/passenger.load), [dashboard.conf](https://github.com/WikiEducationFoundation/WikiEduDashboard/blob/master/server_config/dashboard.conf)  and [dashboard-testing.conf](https://github.com/WikiEducationFoundation/WikiEduDashboard/blob/master/server_config/dashboard-testing.conf). [Database backup script](https://github.com/WikiEducationFoundation/WikiEduDashboard/blob/master/server_config/outreach_mysql_backup.sh) 
* Systemd services for the app's sidekiq processes: https://github.com/WikiEducationFoundation/WikiEduDashboard/tree/master/server_config/systemd
