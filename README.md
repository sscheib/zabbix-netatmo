# zabbix-netatmo

## python/

## php/netatmo.php

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