---
title: RoonLabs music player
description: Instructions on how to integrate a RoonLabs multi room player into Home Assistant.
ha_category:
  - Media Player
ha_release: 0.115
ha_iot_class: Local Push
ha_config_flow: true
ha_codeowners:
  - '@pavoni'
ha_domain: roon
---

The Roon integration allows you to control [RoonLabs](https://roonlabs.com/) music players from Home Assistant.

This integration uses Roon Core, a Roon application that runs on a machine on your network. Via Roon Core, Home Assistant can control all the Roon music players on your network.

To integrate with Roon, you need to provide Home Assistant with the Hostname or IP address of the machine that runs your Roon Core, and then authorize Home Assistant in the Roon application.

If you use an IP address, please assign a static IP address to the machine that runs Roon Core. This ensures that it won't change IP addresses, so you won't have to change the configuration in Home Assistant if it reboots and changes IP address. See your router's manual for details on how to set this up.

## Configuration

You need the Hostname or IP address of the machine that runs your Roon Core. This might be a machine name (which can be followed by `.local`, e.g., `myserver.local`) or can be an IP address.

1. From the Home Assistant front-end, navigate to **Configuration** then **Integrations**. Under **Set up a new integration** locate 'Roon' and click **Configure**.
2. Enter the `Hostname` or `IP address` for the Roon Core machine and click **Submit**.
3. Home Assistant will then contact your Roon Core and ask to be authorized. You will need to enable this extension in the Room Application. Go to **Settings** and then **Extensions**, there you will see an entry for Home Assistant with a button next to it. Click **Enable**.
4. Roon core will then provide Home Assistant with the details of your media players.
5. In Home Assistant you can then pick an area for each of your music players, and add them to Home Assistant.

## Services

#### Service `media_player.play_media`

Roon uses a path based on the roon browser hierarchy to specify which media to play. You can find this by using the media browser, or by following the examples below. If roon can't follow the path you will find an error in the log that will show which part of the path roon could not follow, and the possibilities at that point.

| Service data attribute | Optional | Description                                                                                                                                                            |
| -----------------------| -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `entity_id`            |      yes | Target a specific media player. To target all media players, use `all`.                                                                                                                       |
| `media_content_id`     |       no | A path to specify the media you want to play, see examples below.                   |
| `media_content_type`   |       no | Only `music` is suppported  |

 For example to play the album Harvest by Neil Young you should set `media_content_id` to `Library/Artists/Neil Young/Harvest` and to play BBC Radio 4 you would set `media_content_id` to `My Live Radio/BBC Radio 4`

### Service `roon.join`

Group players together.

| Service data attribute | Optional | Description |
| ---------------------- | -------- | ----------- |
| `entity_id` | yes | ID of the player that will be the master of the group.
| `join_ids` | no | id(s) of the players that will join the master.

### Service `roon.unjoin`

Remove players from a group.

| Service data attribute | Optional | Description |
| ---------------------- | -------- | ----------- |
| `entity_id` | yes | ID of the player that is the master of the group.
| `unlink_name` | yes | id(s) of the players that will be unjoined from the group. If not specified, all players will be unjoined from the master.

### Service `roon.transfer`

Transfer playback from one player to another.

| Service data attribute | Optional | Description |
| ---------------------- | -------- | ----------- |
| `entity_id` | yes | id of the source player.
| `transfer_name` | no | id of the destination player.
