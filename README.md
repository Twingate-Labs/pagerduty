# PagerDuty + ***Twingate*** Integration Benefits

* Notify on-call responders based on the errors reported by Twingate
* Monitor the availability of Twingate resources

# How it Works

* Any error messages reported by the Twingate connector will send an event trigger to a service in PagerDuty
* The error messages from the same resource can be grouped by PagerDuty [Intelligent Alert Grouping](https://support.pagerduty.com/docs/intelligent-alert-grouping)


# Requirements

* A Twingate account ([free to signup](https://auth.twingate.com/signup/?utm_source=pagerduty&utm_medium=partner&utm_campaign=integrations))
* A deployed Twingate connector
* PagerDuty integrations require an [Admin base role](https://support.pagerduty.com/docs/user-roles) for account authorization. If you do not have this role, please reach out to an Admin or Account Owner within your organization to configure the integration.

# Support

If you need help with this integration, please open an issue at the [Twingate Labs - Twingate PagerDuty](https://github.com/Twingate-Labs/twingate-pagerduty) GitHub repository.

# Integration Walkthrough

1. From the **Configuration** menu, select **Services**.
2. There are two ways to add an integration to a service:
    * **If you are adding your integration to an existing service**: Click the **name** of the service you want to add the integration to. Then, select the **Integrations** tab and click the **New Integration** button.
    * **If you are creating a new service for your integration**: Please read our documentation in section [Configuring Services and Integrations](https://support.pagerduty.com/docs/services-and-integrations#section-configuring-services-and-integrations) and follow the steps outlined in the [Create a New Service](https://support.pagerduty.com/docs/services-and-integrations#section-create-a-new-service) section, selecting Twingate as the **Integration Type** in step 4. Continue with the **In Twingate** section (below) once you have finished these steps.
4. Click the **Add Integration** button to save your new integration. You will be redirected to the Integrations tab for your service.
5. An **Integration Key** will be generated on this screen. Keep this key saved in a safe place, as it will be used when you configure the integration with Twingate in the next section.

## In Twingate Connector Server
Enable Twingate connector analytics logging:
-   Add the line line `TWINGATE_LOG_ANALYTICS=v1` in file `/etc/twingate/connector.conf`
-   Restart Twingate Connector `sudo service twingate-connector restart`

Modify and execute the line below to construct the configuration file
```
sudo echo -e 'PAGERDUTY_INTEGRATION_URL={Your Integration URL Here}' > /etc/twingate/twingate-pagerduty.conf
```

Execute following commands to create monitor bash script and service file
```
sudo echo -e '#!/bin/bash\nCONTENT_TYPE="application/json"\njournalctl -u twingate-connector -f -n 0 | \\\nwhile read line ; do\n        echo "$line" | grep \"error_message\"\n        if [ $? = 0 ]\n        then\n                log="${line##*ANALYTICS }"\n                curl -H ${CONTENT_TYPE} -X POST -d "$log" ${PAGERDUTY_INTEGRATION_URL}\n        fi\ndone' > /usr/bin/twingate-pagerduty.sh

sudo chmod +x /usr/bin/twingate-pagerduty.sh

sudo echo -e '[Unit]\nDescription=Twingate PagerDuty Monitor Integration	\nAfter=network-online.target\n\n[Service]\nType=simple\nExecStart=/usr/bin/twingate-pagerduty.sh\nEnvironmentFile=/etc/twingate/twingate-pagerduty.conf\n\n[Install]\nWantedBy=multi-user.target' > /etc/systemd/system/twingate-pagerduty.service

sudo systemctl daemon-reload 
```


Start the twingate-pagerduty service
```
sudo service twingate-pagerduty start
```

# How to Uninstall

1. Navigate to **Services** **Service Directory** select or search for the **service** with the integration you wish to delete.
2. Select the **Integrations** tab to the right of the integration you wish to delete.
3. On the right side, select **Delete Integration**.
4. Confirm your selection in the dialog window.

## In Twingate Connector Server
Stop twingate-pagerduty service
```
sudo service twingate-pagerduty stop
```

Remove configuration, service and script
```
sudo rm -f /etc/twingate/twingate-pagerduty.conf /usr/bin/twingate-pagerduty.sh /etc/systemd/system/twingate-pagerduty.service
```