---
title: Motion Blinds
description: Instructions on how to integrate Motion Blinds from Coulisse B.V. into Home Assistant.
ha_category:
  - Cover
ha_iot_class: Local Push
ha_release: 2020.12
ha_domain: motion_blinds
ha_codeowners:
  - '@starkillerOG'
ha_config_flow: true
---

The integration allows you to control [Motion Blinds](https://motion-blinds.com) from [Coulisse B.V.](https://coulisse.com/products/motion).

To add a Motion Bridge to your installation, click Configuration in the sidebar, then click Integrations. Click the + icon in the lower right. Then search for "Motion Blinds" and enter the setup.

You will be asked for the IP address of the Motion Bridge, e.g., 192.168.1.100. Note that a static IP address is required in order for this integration to keep working properly.

## Retrieving the API Key


The Motion Blinds API uses a 16 character key that can be retrieved from the official "Motion Blinds" app for [IOS](https://apps.apple.com/us/app/motion-blinds/id1437234324) or [Android](https://play.google.com/store/apps/details?id=com.coulisse.motion).

Open the app, click the 3 dots in the top right corner, go to "settings", go to "Motion APP About", Please quickly tap this "Motion APP About" page 5 times, a popup will appear that gives you the key.

Please note that "-" characters need to be included in the key when providing it to Home Assistant. The key needs to be similar to `12ab345c-d67e-8f`

<p class='img'>
<img src='/images/integrations/motion_blinds/Motion_App__get_key_1.jpg' />
<img src='/images/integrations/motion_blinds/Motion_App__get_key_2.jpg' />
</p>

## Top Down Bottom Up (TDBU) blinds

TDBU blinds consist of two bars controlled by two motors designated by Top and Bottom with fabric in between.
The Top and Bottom can move independently of each other to cover different parts of the window.
Controlling the two bars can be done through three different entities that will be created: Top, Bottom and Combined.

### Top entity

- 'Up/Open' will move the Top bar to the top of the window (absolute position 100).
- 'Down/Close' will move the Top bar to the position of the Bottom bar, therefore, making the part of the window that is covered as small as possible, but the two bars will be at the position of the Bottom bar (not at the top of the window).
- 'Position' is the relative position in which the Top bar can move, so from the top of the window (100) to the position of the Bottom bar (0), note that the position will therefore change if the Bottom bar is moved, since the space in which the Top bar is allowed to move changes.
- 'Absolute position' is the position of the Top bar with respect to the window, so 0 = bottom of the window and 100 = top of the window. Note that not all absolute positions are reachable at all moments due to the Bottom bar.
- 'Width' is the percentage of the window covered by fabric (the space between the Top and Bottom bars).

### Bottom entity

- 'Up/Open' will move the Bottom bar to the position of the Top bar.
- 'Down/Close' will move the Top bar to the bottom of the window (absolute position 0).
- 'Position' is the relative position in which the Bottom bar can move, so from the position of the Top bar (100) to the bottom of the window (0), note that the position will therefore change if the Top bar is moved, since the space in which the Bottom bar is allowed to move changes.
- 'Absolute position' is the position of the Bottom bar with respect to the window, so 0 = bottom of the window and 100 = top of the window. Note that not all absolute positions are reachable at all moments due to the Top bar.
- 'Width' is the percentage of the window covered by fabric (the space between the Top and Bottom bars).

### Combined entity

- 'Up/Open' will move both the Top and Bottom bars to the top of the window, effectively covering as little as possible of the window (Width will be 0 %).
- 'Down/Close' will move the Top bar to the top of the window and the Bottom bar to the bottom of the window, effectively covering the whole window (Width will be 100 %).
- 'Position' is the relative position of the center between the Bottom and Top bars in which the center can move, so basically, such that the area covered by the Bottom and Top bar can be moved without changing its size, such that the Top bar can go to the top of the window and the Bottom bar to the bottom of the window.
- 'Absolute position' is the position of the center between the Bottom and Top bars with respect to the window, so 0 = bottom of the window and 100 = top of the window. Note that not all absolute positions are reachable at all moments due to the width.
- 'Width' is the percentage of the window covered by fabric (the space between the Top and Bottom bars).

### TDBU notes

Because the relative position is used by default in Home Assistant, scenes will not work properly with TDBU blinds (depending on the current position of the Top en Bottom, a certain position (70) will correspond to a different position with respect to the window).

Therefore it is recommended to use scripts or automations with the TDBU Combined entity that use the `motion_blinds.set_absolute_position` with specifying both the `absolute_position` and `width`.

That will ensure the same absolute position with respect to the window is achieved without letting the Bottom or Top bar move to an absolute_position that is not allowed.

When the `motion_blinds.set_absolute_position` service is used with values that would move the Bottom or Top bar to positions that will make them collide, nothing will happen. An error will be logged telling that that position is not allowed and the TDBU blind will not move.

Therefore it is always safe to use any of the services in Home Assistant with the TDBU blinds.

## Service `motion_blinds.set_absolute_position`

For most blinds the `motion_blinds.set_absolute_position` does the same as `cover.set_cover_position` service.
However, for TDBU blinds it will set the absolute position relative to the window itself.
The `cover.set_cover_position` will set the scaled position relative to the space in which the TDBU blind is allowed to move.

| Service data attribute | Optional | Description                                                                                       |
| ---------------------- | -------- | ------------------------------------------------------------------------------------------------- |
| `entity_id`            |      yes | Name of the motion blind cover entity to control. For example `cover.TopDownBottomUp-Bottom-0001` |
| `absolute_position`    |       no | Absolute position to move to. For example 70                                                      |
| `width`                |      yes | Optionally specify the width that is covered, only for TDBU Combined entities. For example 30     |
