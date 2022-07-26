# Data Cleansing

This rule implements +1 processing of the data collected by the neuron from the device, renames it to a meaningful name, and sends the result to the MQTT dynamic topic `${node_name}/${group_name}` in the cloud.

## Step 1 Add a new rule

Click `New Rule` to create a new rule in the rule page, as shown below.

![data-stream-rules-add](./assets/data-stream-rules-add.png)

## Step 2 Setup rule details

In the rule editing interface, fill in the rule information, as shown below.

![data-stream-rules-add-action](./assets/data-stream-rules-add-action.png)

* Fill in the `Rule ID` and `SQL` statement.
* Click on `Add` button to add sink action for the rule, you may add more than one sink action for each rule, see step 3 for details.
* Click on `Submit` button to complete the rule definition.

## Step 3 Setup sink details

Set sink details in the pop-up window for adding actions, as shown below.

![data-stream-rules-action](./assets/data-stream-rules-action.png)

::: v-pre

* Drop down to select sink, e.g select mqtt.
* Fill in the MQTT broker address.
* Fill in the MQTT topic, in this case, we have `{{.node_name}}/{{.group_name}}`.
* Select the `True` for send single.
* Click on `Submit` button to complete the sink action.
:::

After the action is added, as shown below.

![data-stream-rules](./assets/data-stream-rules.png)

## Step 4 Start rule execution

Start the rule, as shown below.

![data-stream-rules-list](./assets/data-stream-rules-list.png)

## Step 5 Check over the payload in MQTT broker

::: v-pre

Start MQTTX client, subscribe the topic `{{.node_name}}/{{.group_name}}`, the result is shown below.
:::

::: tip
The node_name used in this example is **modbus-plus-tcp-1** and the group_name is **group-1**, that is, the subscription topic is modbus-plus-tcp-1/group-1.
:::

![result](./assets/result.png)