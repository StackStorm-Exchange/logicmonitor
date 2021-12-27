# LogicMonitor Integration Pack

Pack which allows integration with your company's [LogicMonitor](https://www.logicmonitor.com/)
Portal.

## Configuration File

Copy the example configuration in [logicmonitor.yaml.example](./logicmonitor.yaml.example)
to `/opt/stackstorm/configs/logicmonitor.yaml` and edit as required.

It must contain:

* ``company`` - The name of your company as seen in your LogicMonitor portal's url.<br/>
  <br/>For example, if your company's name is "Example Company" and you access your portal at
  example.logicmonitor.com then this field should be "example".<br/><br/>
* ``access_id`` - Your LogicMonitor portal's API Access ID.<br/>
  > <b>NOTE</b>: The LogicMonitor Pack includes a set of Actions that make REST requests to your LogicMonitor Portal using the [LogicMonitor Python SDK](https://www.logicmonitor.com/support-files/rest-api-developers-guide/sdks/docs/). Therefore, it requires a valid [LogicMonitor API Access ID and Access Key](https://www.logicmonitor.com/support/settings/users-and-roles/api-tokens).
  > <br/><br/>You can create an Access ID and Access Key Pair in your LogicMonitor Portal by going to <b>
  Settings -> Users & Roles -> API Tokens -> LMv1 -> Add</b>.<br/>
* ``access_key`` <b>(SECRET) </b> - Your LogicMonitor portal's API Access Key.<br/>
  > <b>WARNING:</b> `access_key` is a secret value so don't save it in `/opt/stackstorm/configs/logicmonitor.yaml`
  > directly as clear text. Instead, use StackStorm's [dynamic configuration values](https://docs.stackstorm.com/reference/pack_configs.html#configuring-a-user-scoped-dynamic-configuration-value) to populate the `access_key` field.
  > <br/>
  > <br/>
  > <b>NOTE:</b> When using the `st2 key set key_name key_value` command to create a dynamic configuration value:<br/>
  > * Be sure to encrypt the value by using the `--encrypt` flag.
  > * Use `--` after the last flag to indicate to the compiler the end of flag-processing. This guarantees that all API Access Keys generated by LogicMonitor will be accepted by the command.
  >
  >
  > The final command to generate a dynamic configuration value should look something like this:
  > <br/>
  > `st2 key set --scope=user --encrypt -- lm_access_key "{k%J7I(Gkf^5sgH8tdT=85485fX-}V2z4gkCfkPH"`
  > <br/><br/>Once a dynamic configuration value has been created you must reference it in your configuration file using `"{{st2kv.user.key_name}}"`.
  > <br/><br/>Read the [documentation](https://docs.stackstorm.com/reference/pack_configs.html#configuring-a-user-scoped-dynamic-configuration-value) to ensure you understand how dynamic configuration values work.

  > <b>NOTE</b>: The LogicMonitor Pack includes a set of Actions that make REST requests to your LogicMonitor Portal using the [LogicMonitor Python SDK](https://www.logicmonitor.com/support-files/rest-api-developers-guide/sdks/docs/). Therefore, it requires a valid [LogicMonitor API Access ID and Access Key](https://www.logicmonitor.com/support/settings/users-and-roles/api-tokens).
  > <br/><br/>You can create an Access ID and Access Key Pair in your LogicMonitor Portal by going to <b>
  Settings -> Users & Roles -> API Tokens -> LMv1 -> Add</b>.<br/>
* ``auth_enabled`` - True or false, defaults to <b>true</b>.
  <br/><br/>If enabled, this authenticates all requests made to StackStorm using
  the [StackStorm API Key](https://docs.stackstorm.com/authentication.html#api-keys)
  entered into your StackStorm Integration in your LogicMonitor portal. Set to false to make testing
  easier.
  > <b>WARNING:</b> We strongly recommend setting `auth_enabled` to <b>true</b> for production.

**Note** : When modifying the configuration in `/opt/stackstorm/configs/` please remember to tell
StackStorm to load these new values by running
`st2ctl reload --register-configs`

## LogicMonitor REST API Access ID and Key

As mentioned, the LogicMonitor Pack comes with a number of Actions that make REST requests to your
LogicMonitor Portal using
the [LogicMonitor Python SDK](https://www.logicmonitor.com/support-files/rest-api-developers-guide/sdks/docs/)
. Therefore, it requires a
valid [LogicMonitor API Access ID and Access Key](https://www.logicmonitor.com/support/settings/users-and-roles/api-tokens)
.

You can create an Access ID and Access Key Pair in your LogicMonitor portal by going to <b>
Settings -> Users & Roles -> API Tokens -> LMv1 -> Add</b>.<br/>

As discussed further above, you must enter a valid LogicMonitor API Access ID and Access Key Pair
into your `/opt/stackstorm/configs/logicmonitor.yaml` configuration file.

* The Access ID is entered into the configuration file's `access_id`field.
* The Access Key is a secret that is should be entered into the configuration file's `access_key`
  field using
  StackStorm's [dynamic configuration values](https://docs.stackstorm.com/reference/pack_configs.html#configuring-a-user-scoped-dynamic-configuration-value)
  . The access key is secret and should never be exposed.

## Sensor & StackStorm API Key

The LogicMonitor Pack launches a sensor on port 5000 that authenticates every request with
StackStorm using a [StackStorm API Key](https://docs.stackstorm.com/authentication.html#api-keys).

The StackStorm API Key must be generated in StackStorm and then copy/pasted into your LogicMonitor
StackStorm Integration's "StackStorm Api Key" field.

To create a new StackStorm Integration in your LogicMonitor Portal go to <b>Settings -> Integrations
-> Add -> StackStorm</b>.

> <b>NOTE:</b> The StackStorm API Key is stored in the "apiKey" object in the payload of the POST Request sent out by your LogicMonitor StackStorm Integration. Therefore, <b><u>the "apiKey" object must exist in the payload</u></b>. If you remove the "apiKey" object from the payload authentication with StackStorm will fail.<br/><br/>You can view and modify the POST Request's payload at the bottom of the StackStorm Integration dialog.

## Sensor & Port 5000 & Networking

The LogicMonitor Pack launches a sensor (a Flask server) on port 5000. Be sure to modify your
network settings to allow internet traffic to reach port 5000 on the machine on which StackStorm has
been installed.

## Sensor -> Trigger -> Rule -> Action

Once an alert payload has been successfully authenticated with StackStorm, the sensor will inject
the
`logicmonitor.alert_trigger` trigger with the alert payload into StackStorm. The purpose of
injecting the trigger with the concomitant payload is to fire a custom Action if the conditions in a
Rule are satisfied. In that regard, the LogicMonitor Pack has provided a number of Rules (disabled
by default) that fire an Action based on the type of alert being sent:

* `alert_throttling_alert_rule.yaml` - Maps Alert Throttling Alerts (LMT) to an action
* `collector_failover_alert_rule.yaml` - Maps Collector Failover Alerts (LMF) to an action
* `collector_failover_unreachable_alert_rule.yaml` - Maps Collector Failover Unreachable Alerts (
  LMF) to an action
* `collector_unreachable_alert_rule.yaml` - Maps Collector Unreachable Alerts (LMA) to an action
* `configsource_alert_rule.yaml` - Maps ConfigSource Alerts (LMD) to an action
* `datasource_alert_rule.yaml` - Maps DataSource Alerts (LMD) to an action
* `devicegroup_alert_rule.yaml` - Maps DeviceGroups Alerts (LMHC) to an action
* `eventsource_alert_rule.yaml` - Maps EventSource Alerts (LME) to an action
* `jobmonitor_alert_rule.yaml` - Maps JobMonitor Alerts (LMB) to an action
* `lmlogs_alert_rule.yaml` - Maps LM Log Alerts (LML) to an action
* `service_alert_rule.yaml` - Maps Service Alerts (LMS) to an action

> <b>NOTE:</b> All of these Rules are disabled by default. Enable them inside the Rule's <i>.yaml</i> file if you wish to use them.

> <b>NOTE:</b> All of these Rules fire the `get_alert_list` dummy action by default. Be sure to modify the Action that gets fired when a Rule's criteria is satisfied by changing the Action reference inside that Rule's <i>.yaml</i> file.

## Available Actions

The LogicMonitor Pack also comes with a large (but not exhaustive) number of Actions that make their
corresponding REST Requests to your LogicMonitor Portal. On a high level, the list of actions
currently in the pack are:

* `Acknowledge Alert By ID`
* `Get Alert By ID`
* `Get Alert List`
* `( Add, Delete, Get, Patch ) x Admin`
* `( Add, Delete, Get, Patch ) x Alert Rule`
* `( Add, Delete, Get, Patch ) x Collector`
* `( Add, Delete, Get, Patch ) x Device`
* `( Add, Delete, Get, Patch ) x Device Group`
* `( Add, Delete, Get, Patch ) x Escalation Chain`
* `( Add, Delete, Get, Patch ) x Ops Note`
* `( Add, Delete, Get, Patch ) x SDT`

More specifically, the LogicMonitor Pack has provided these Actions:

###### GENERAL

* `ack_alert_by_id`
* `ack_collector_down_alert_by_id`

###### ADD

* `add_admin`
* `add_alert_rule`
* `add_collector`
* `add_device`
* `add_device_group`
* `add_escalation_chain`
* `add_ops_note`
* `add_sdt`

###### DELETE

* `delete_admin_by_id`
* `delete_alert_rule_by_id`
* `delete_collector_by_id`
* `delete_device_by_id`
* `delete_device_group_by_id`
* `delete_escalation_chain_by_id`
* `delete_ops_note_by_id`
  `delete_sdt_by_id`

###### GET

* `get_admin_by_id`
* `get_admin_list`
* `get_alert_by_id`
* `get_alert_list`
* `get_alert_rule_by_id`
* `get_alert_rule_list`
* `get_collector_by_id`
* `get_collector_list`
* `get_device_by_id`
* `get_device_group_by_id`
* `get_device_group_list`
* `get_device_list`
* `get_escalation_chain_by_id`
* `get_escalation_chain_list`
* `get_ops_note_by_id`
* `get_ops_note_list`
* `get_sdt_by_id`
* `get_sdt_list`

###### PATCH

* `patch_admin_by_id`
* `patch_alert_rule_by_id`
* `patch_collector_by_id`
* `patch_device`
* `patch_device_group_by_id`
* `patch_escalation_chain_by_id`

> <b>NOTE:</b> All of these Actions correspond to REST functions that exist in the [LogicMonitor Python SDK](https://www.logicmonitor.com/support-files/rest-api-developers-guide/sdks/docs/) of the same name. For example, the `patch_escalation_chain_by_id` Action will fire the [`patchEscalationChainById` function in the LM Python SDK](https://www.logicmonitor.com/support-files/rest-api-developers-guide/sdks/docs/#api-LM-patchEscalationChainById).

> <b>NOTE:</b> All of these Actions call the same <i>run.py</i> method but with different parameters.

> <b>NOTE:</b> This is not an exhaustive list of actions that are available in the [LogicMonitor Python SDK](https://www.logicmonitor.com/support-files/rest-api-developers-guide/sdks/docs/).
>
> If you need to use a function from the LM Python SDK but it hasn't been included in the pack, copy/paste an existing action and modify it to call the corresponding function from the LM Python SDK. Be sure to refer to the [documentation](https://www.logicmonitor.com/support-files/rest-api-developers-guide/sdks/docs/) to see the list of parameters needed for the specific function you intend on adding.

## Thank you for downloading the LogicMonitor Pack!
