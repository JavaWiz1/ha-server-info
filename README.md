# ha-server-info

### Home Assistant Server Information Template (for use with Glances)

![Example Dashboard](images/Dashboard.png)

Home networks have expanded to include a wide variety of devices over the years.  In particular, windows desktops, laptops, Linux Boxes, NAS servers, Raspberry PI's, etc...

Keeping track of these cross-platform devices and their information has become a challenge.  Glances is an open-source cross platform monitoring tool that provides a htop (for you linux fans),
or task-monitor (for you windows users) interface to keep track of system cpu usages, disk/memory usage, tasks, etc...

This template is a HA Card that displays device information for target hosts.  You may tweak the template to render the stats important to you, or
use out-of-the box to display stats as shown in the moc-ups.


# Requirements
This repository leverage:

- [Glances](https://github.com/nicolargo/glances) for system information 
- Home Assistant [decluttering-card](https://github.com/custom-cards/decluttering-card) Lovelace interface 

to provide a handy, easy-to-use template for displaying system information for windows, linux, raspberry pi systems.

# Pre-requisite Installs
## Glances

[Glances](https://github.com/nicolargo/glances) is a python app that runs on target servers.  It gathers runtime information (cpu, disk, memory, processes, etc..) that is exposed via API to Home Assistant and is represented as
sensor data.

### Install
Refer to the [install directions](https://github.com/nicolargo/glances?tab=readme-ov-file#installation) for Glances.
- I use pipx for the installation, but pip/pip3 works also.
- At a minimum, the [web] feature needs to be installed (i.e. pipx install glances[web])
- I personally have tested it on Raspi OS, Ubuntu, Windows 10 and Windows 11.  
- The Windows web api seems to be a little intermittent responding, so sometimes server information is not available.

### Configure

To configure targets:
- Go to the integration for Glances (Setting -> Devices and Services -> Glances)
- Click "Add Entry"
- Enter Host name
  - Note this was a bit tricky for me.  The short name didn't work, I had to fully qualify with a domain (in my case workgroup) name (i.e. MyLaptop.mylan)
- Click submit.  If it works, your device will be added, and entities will auto-populate with sensor data.
  
---

---

## decluttering-card

The [decluttering-card](https://github.com/custom-cards/decluttering-card) is a Home Assistant Lovelace helper which provides the ability to build a re-usable template (card) to display information on your Home assistant dashboard, greatly reducing yaml bloat and making it easier to provide consistent UI updates and features.

### Install

The [install instructions](https://github.com/custom-cards/decluttering-card#installation) show how to make the decluttring card available for your dashboards.

Installing the card by above instructions didn't work for me, so I installed/downloaded via HACS.
- Insure HACS is installed in HA.
- In HACS, search for declutting-card
- In the row displayed, click the three dots ... on right side.
- Click download to install the requisite .js file

It basically copies a javascript file (decluttering-card.js) to your Lovelace www directory (config/www/community/decluttering-card).

# Usage

## Validate Pre-requisites
- **Install Glances** on target machines (see above).  Be sure the service is started with the -w (web server) option.
  - Validate service is running by going to http://\<target machine\>:61208 and insure page is displayed.
- **Configure Glances** in your HA instance (see above).
  - Validate sensors are available (Setting -> Devices and Services -> Glances)
  - Insure devices are defined and entities are available.
  - Note the ***device name*** in the Entity ID, this will be used to create your card.
- **Install decluttering-card** in your Home Assistant instance (see above).

## Create a dashboard
- Settings -> Dashboards
- Click Add dashboard (new dashboard from scratch)
  - Give it a meaningful title (i.e. LAN Hosts)
  - Click Create
- Open the dashboard
  - Click the pencil icon (top right) to edit the dashboard
  - Click the three dots (...) top right and select 'Raw configuration editor'
  You should see the below in the editor
  ```
  views:
  - title: LAN Hosts
  ```
- Copy the contents of [decluttering_template.yaml](https://raw.githubusercontent.com/JavaWiz1/ha-server-info/refs/heads/develop/decluttering_templates.yaml) to the top of the file in the editor (before the content above).  Beware of the indentation.

Your view should reflect below (... are collapsed sections)
```
decluttering_templates:
  glances_windows_device:
    ...
  glances_rpi_device:
    ...
  glances_linux_device: 
    ...
views:
  - title: LAN Hosts
```

You now have an empty dashboard with 3 available templates.  

- glances_windows_device
- glances_rpi_device
- glances_linux_device
  
This represents a 'shell' dashboard.  You may now add content as desired (see example below).

## Create host cards for content

Each host is represented by a type block as follows:
```
    - type: grid
      cards:
        - type: custom:decluttering-card
          template: glances_XXX_device
          variables:
            - host: hostname
            - title: Friendly Name
```

- The **template** is glances_rpi_device OR glances_windows_device OR glances_linux_device depending on target.
- The **host** is the full host name as shown in the entity_id (Setting -> Devices and Services -> Glances).
- The **title** is the friendly name you want displayed in the UI.
 
# Example

Assume 

- Glances is setup and configured to monitor server1 (raspberry pi) and server2 (windows machine)
- The decluttering-card has been successfully installed either manually or via HACS.

To create cards on this dashboard, edit the dashboard via the Raw configuration editor by adding section below to the views section:
```
    sections:
      - type: grid
        cards:
          - type: custom:decluttering-card
            template: glances_rpi_device
            variables:
              - host: server1_local
              - title: My Rasberry Pi
      - type: grid
        cards:
          - type: custom:decluttering-card
            template: glances_windows_device
            variables:
              - host: server2_local
              - title: My Windows Laptop
```

Full editor view with sections in decluttering_templates collapsed:

```
decluttering_templates:
  glances_windows_device:
    ...
  glances_rpi_device:
   ...
  glances_linux_device: 
    ...
views:
  - title: My LAN 
    sections:
      - type: grid
        cards:
          - type: custom:decluttering-card
            template: glances_rpi_device
            variables:
              - host: server1_local
              - title: My Raspberry Pi
      - type: grid
        cards:
          - type: custom:decluttering-card
            template: glances_windows_device
            variables:
              - host: server2_local
              - title: My Windows Laptop
```
NOTE: Change host and title variables to reflect your setup.

Save your changes and Click Done.  The dashboard should now look similar to:

![NyLan Dashboard](images/MyLanDashboard.png)

To add additional hosts, simply continue to add the appropriate blocks for each host via the Raw configuration editor.

