# zabbix-netatmo

## python version

Should work with python 2.7 or 3.x
Requires python-requests and python-six

Note : netatmo_standalone.py is the legacy and deprecated version which wont support items discovery

Rename config.ini.example as config.ini and edit variables in the [main] section :
* device_id = Your device id (see below), or leave empty in order to get data for every station configured on your account
* client_id = a client id generated on https://dev.netatmo.com/
* client_secret = the correspondant client secret generated on https://dev.netatmo.com/

Import the template into your zabbix instance, and add it to the desired host.

On first use, you will need to initialize your tokens using the grant.py script (this will also prompts you for your netatmo username and password) :
```
#$ python grant.py
```

You can then try the main script in order to see which items are generated:
```
#$ python netatmo.py
```

In order to send those data to your zabbix instance, you need to have a properly configured zabbix agent configuration file and zabbix_sender binary intalled (debian package zabbix-sender)
```
#$ python netatmo.py discovery | zabbix_sender -c /etc/zabbix/zabbix_agentd.conf -i -
#$ #wait a minute or so, zabbix sometime takes times before creating every discovered items
#$ python netatmo.py | zabbix_sender -c /etc/zabbix/zabbix_agentd.conf -i -
```
"failed" number should be 0. If otherwise, check your zabbix items configuration


If everything is working accordingly, use your system crontab to send data automatically to your zabbix server on a regular basis :
```
0    4 * * * python /path/to/netatmo.py discovery | zabbix_sender -c /etc/zabbix/zabbix_agentd.conf -i - > /dev/null
*/5  * * * * python /path/to/netatmo.py | zabbix_sender -c /etc/zabbix/zabbix_agentd.conf -i - > /dev/null
```

### Obtaining your Device ID

To find your Device ID, navigate to the netatmo dashboard (https://my.netatmo.com/app/station) and select the settings icon (gear icon on the top right) and scroll down until you find the main Indoor Module (the one which have wifi signal info). The device ID is this module MAC Address. It should have the pattern 00:00:00:00:00:00, with letters or numbers in each two-character set.

### Todo

* value mapping for wifi_status, rf_status, battery_status as per https://dev.netatmo.com/en-US/resources/technical/reference/weatherstation and https://dev.netatmo.com/en-US/resources/technical/reference/weatherstation/getstationsdata (waiting for https://support.zabbix.com/browse/ZBXNEXT-114)

## php version (deprecated, ugly)

require php5-curl and zabbix_sender

clone netatmo PHP api and place netatmo.php inside the cloned directory :
```
#$ git clone https://github.com/Netatmo/Netatmo-API-PHP.git
#$ mv php/netatmo.php Netatmo-API-PHP/
```

configure Netatmo-API-PHP/netatmo.php :
```
$config['client_id'] = "client-id-from-dev.netatmo.com";
$config['client_secret'] = "client-secret-from-dev.netatmo.com";
$username = "your-netatmo-login";
$pwd = "your-netatmo-password";
```

run it in order to see which items are generated :
```
#$ php Netatmo-API-PHP/netatmo.php data
```

Note that this script wont work if your netatmo station name or if any of your netatmo module have a space in its name

Your main netatmo station will be called "main"
netatmo.StationName.main
All other netatmo module will be given their own name
netatmo.StationName.ModuleName

Add provided items to your zabbix hosts config

Use zabbix_sender to provide firsts data to your server
```
#$ netatmo.php data | zabbix_sender -c /etc/zabbix/zabbix_agentd.conf -i -
```
"failed" number should be 0. If otherwise, check your zabbix items configuration

Use crontab + zabbix-sender to send data to your zabbix server :
```
/usr/bin/php /path/to/Netatmo-API-PHP/netatmo.php data | zabbix_sender -c /etc/zabbix/zabbix_agentd.conf -i - > /dev/null
```
