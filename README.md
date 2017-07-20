# hassctl <a href="https://flattr.com/submit/auto?fid=o7dr10&url=https%3A%2F%2Fgithub.com%2Fdale3h%2Fhassctl" target="_blank"><img src="https://button.flattr.com/flattr-badge-large.png" alt="Flattr this" title="Flattr this" border="0"></a>

A command line utility to simplify the management of Home Assistant

## Compatibility

This utility has been tested on the following platforms:

* Raspberry Pi
  * Manual install (with `systemd` service)
  * AIO/One-Step Installer
  * HASSbian Image
* Ubuntu Server 16.04.1
  * Manual install (with `systemd` service)

## Installation

`sudo curl -o /usr/local/bin/hassctl https://raw.githubusercontent.com/dale3h/hassctl/master/hassctl && sudo chmod +x /usr/local/bin/hassctl`

## Updating Home Assistant

The safest way to update Home Assistant in a production environment is to run:

`hassctl update-hass && hassctl config && hassctl restart`

This set of commands will update Home Assistant, run a configuration check, and then restart.

If the update fails, the configuration check will not run.

If the configuration check fails, Home Assistant will not be restarted.

If you would like to install a specific version of Home Assistant, run:

`hassctl update-hass 0.43.1` (replace `0.43.1` with the version you wish to install)

## Updating `hassctl`

To update `hassctl` to the latest stable version, run `hassctl update-hassctl` or `hassctl update-hassctl master`

To update `hassctl` to a custom branch on this repo, run `hassctl update-hassctl branch-name`

## Usage

### You can update Home Assistant using:

**`$ hassctl update-hass`** - Update Home Assistant to the latest version on PyPi

**`$ hassctl update-hass 0.47.0`** - Update Home Assistant to version 0.47.0

### You can control Home Assistant using:

**`$ hassctl start`** - Start the Home Assistant service

**`$ hassctl stop`** - Stop the Home Assistant service

**`$ hassctl restart`** - Restart the Home Assistance service

**`$ hassctl kill`** - Send a SIGKILL (-9) signal to the Home Assistant service

**`$ hassctl log`** - Follow the Home Assistant logs (errors are highlighted)

**`$ hassctl error`** - Follow the Home Assistant error logs

**`$ hassctl debug`** - Follow the Home Assistant debug logs

**`$ hassctl zwave`** - Follow the Open Z-Wave logs

**`$ hassctl config`** - Run the configuration check script

**`$ hassctl update-hassctl [branch]`** - Update `hassctl` to the latest version

## Configuration Examples

The configuration file is located at `/etc/hassctl.conf`.

### HASSbian

```
HASSCTL_BRANCH=master

VIRTUAL_ENV=/srv/homeassistant
PIP_EXEC=$VIRTUAL_ENV/bin/pip3
HASS_EXEC=$VIRTUAL_ENV/bin/hass

HASS_CONFIG=/home/homeassistant/.homeassistant
HASS_USER=homeassistant
HASS_SERVICE=home-assistant@homeassistant.service

OZW_LOG=$HASS_CONFIG/OZW_Log.txt
```

### Current AIO Installer

```
HASSCTL_BRANCH=master

VIRTUAL_ENV=/srv/homeassistant/homeassistant_venv
PIP_EXEC=$VIRTUAL_ENV/bin/pip3
HASS_EXEC=$VIRTUAL_ENV/bin/hass

HASS_CONFIG=/home/homeassistant/.homeassistant
HASS_USER=homeassistant
HASS_SERVICE=home-assistant.service

OZW_LOG=$HASS_CONFIG/OZW_Log.txt
```

### Pre-December 2016 AIO Installer

```
HASSCTL_BRANCH=master

VIRTUAL_ENV=/srv/hass/hass_venv
PIP_EXEC=$VIRTUAL_ENV/bin/pip3
HASS_EXEC=$VIRTUAL_ENV/bin/hass

HASS_CONFIG=/home/hass/.homeassistant
HASS_USER=hass
HASS_SERVICE=home-assistant.service

OZW_LOG=$HASS_CONFIG/OZW_Log.txt
```

## Example YAML Package
hassctl must have sudo access in the sudoers file. ```homeassistant ALL=(ALL) NOPASSWD: /usr/local/bin/hassctl```
```
homeassistant:
  customize:
    script.hassctl_update_ha:
      friendly_name: Update Home Assistant
      icon: mdi:package-variant
    script.hassctl_update:
      friendly_name: Update hassctl
      icon: mdi:package-variant
    script.hassctl_restart_ha:
      friendly_name: Restart Home Assistant
      icon: mdi:restart
    sensor.hassctl_version:
      friendly_name: Version
###### STATE CARD #######################################################
group:
  hassctl:
    name: hassctl
    control: hidden
    entities:
      - script.hassctl_restart_ha
      - script.hassctl_update_ha
      - script.hassctl_update
      - sensor.hassctl_version
#########################################################################
script:
  hassctl_restart_ha:
    sequence:
      - service: shell_command.hassctl_restart_ha
  hassctl_update_ha:
    sequence:
      - service: shell_command.hassctl_update_ha
  hassctl_update:
    sequence:
      - service: shell_command.hassctl_update
#########################################################################
shell_command:
  hassctl_restart_ha: 'sudo hassctl restart'
  hassctl_update_ha: 'sudo bash /home/homeassistant/.homeassistant/shell/update_ha.sh </dev/null >> /home/homeassistant/.homeassistant/update_ha.log 2>&1 &'
  hassctl_update: 'sudo hassctl update-hassctl'
#########################################################################
sensor:
  ###### VERSIONS
    - platform: command_line
      name: hassctl Current Version
      command: curl -s  https://raw.githubusercontent.com/dale3h/hassctl/master/hassctl 2> /dev/null | head -3 | tail -1 | cut -d# -f2
      value_template: '{{value[17:-1]}}'
    - platform: command_line
      name: hassctl Installed Version
      command: cat /usr/local/bin/hassctl | head -3 | tail -1 | cut -d# -f2
      value_template: '{{value[17:-1]}}'
  ###### Single line Version
    - platform: template
      sensors:
        hassctl_version:
          value_template: "{%- if states.sensor.hassctl_current_version.state == states.sensor.hassctl_installed_version.state-%}{{states.sensor.hassctl_installed_version.state}} {% else %}{{states.sensor.hassctl_current_version.state}} Available{% endif%}"
          icon_template: >-
            {% if states.sensor.hassctl_current_version.state == states.sensor.hassctl_installed_version.state %}
              mdi:checkbox-marked
            {% else %}
              mdi:checkbox-blank-outline
            {% endif %}
```
### update_ha.sh
```
#!/bin/bash
hassctl update-hass && hassctl restart
```
