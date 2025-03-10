# Alternative Home Connect Integration for Home Assistant
This project is an alternative integration for Home Connect enabled home appliances manufactured by BSH under the Bosch, Siemens, Constructa and Neff brands.  

</br>

# Main features
Home Assistant already has a built-in integration for Home Connect, however, it is quite basic, generates entities that are not always supported by the connected appliances and tends to stop getting status updates after a while.
This integration attempts to address those issues and has the following features:
* All the entities are dynamically read from the API and reflect true capabilities of the appliance.
* The integration exposes entities that provide complete control over programs, program options, and global settings. These entities are dynamically read from API and therefore are specifically applicable to the connected appliances.
* Configurable options and settings are exposed for easy selection using "Select", "Switch" or "Number" entities, as appropriate.
* Read only status values, as well as some selectable options are also made available either using "Sensor" or "Binary Sensor" entities for easier use when only wanting to display them.
* Status events that are published by the Home Connect service are exposed as Home Assistant events.
* "Program Started" and "Program Finished" events are exposed as triggers for easier building of automation scripts.
* A "Start Program" Button entity is provided to start the operation of the selected program.
* Program and option selections are also available as a service for easier integration in scripts.
* The state of all entities is updated in real time with a cloud push type integration.
* Clean handling of appliances disconnecting and reconnecting from the cloud.
* Clean handling of new appliances being added or removed from the service.
* All the names support translation, but currently only the English translation is provided.
* Using pure async implementation for reduced load on the platform.
  
</br>  

# Installation instruction
## Create a Home Connect app
Before installing the integration you need to create an "application" in the Home Connect developers website. Note that if you have an existing appliacation, that was created before July 2022 you will most likely have to update the redirect URI to the one specified below. It can take a few hours for the changes to existing applications to apply, so be patiant. 
  
1. Navigate to the "[Applications](https://developer.home-connect.com/applications)"
   page on the Home Connect developers website. You'll be prompted to create an account or sign in if you already have one.
2. Click the "[Register Application](https://developer.home-connect.com/applications/add)" link. 
3. Fill in the application creation form:  
   **Application ID**: A unique ID for the application, can be home-connect-alt, or whatever you like.  
   **OAuth Flow**: Authorization Code Grant Flow  
   **Home Connect User Account for Testing**: Leave blank  
   **Redirect URI**: https://my.home-assistant.io/redirect/oauth  
   **Add additional redirect URIs**: Leave unchecked  
   **Enable One Time Token Mode**: Leave unchecked  
4. Click "Save" then copy the *Client ID* and *Client Secret* and save them for use in the next step.

## Update the configuration.yaml file
At a minimum you need to add the following to your configuration.yaml file:
```
home_connect_alt:
  client_id: < Your Client ID >
  client_secret: < You Client Secret >
```
This configuration should be enough for most people. For more avanced options see the [Configuration options](#configuration-options) section below. 

## Install the integraion
1. The esiest way to install the integration is using HACS. Just click the 
   button bellow and follow the instructions:  
  [![Open your Home Assistant instance and open a repository inside the Home Assistant Community Store.](https://my.home-assistant.io/badges/hacs_repository.svg)](https://my.home-assistant.io/redirect/hacs_repository/?owner=ekutner&repository=home-connect-hass)
2. Nagivate to https://my.home-assistant.io/ and make sure the Home Assistant Instance is configured correctly to point to your local Home Assistant instance.
3. Power on all your appliances.
4. Turn **OFF** the wifi on your phone and make sure all the appliances are operational in the Home Connect mobile app.
5. In your Home Assistant instance navigate to Settings -> Devices & Services then clieck the "Add Integration" button. Search for "Home Connect Alt" and install it.
6. A new window will popup where you will be asked to login to to your Home Connect 
   account and allow Home Assistant to access your appliances. After you approve that you will be redirected back to Home Assistant, continue as instructed. 
7. Congratulations, you're done!  
   Home Connect Alt will now start downloading the data for your
   appliances and will add the entities for them to Home Assistant.  
   Note that the integration dynamically discoveres entities as they are made available by the API, so expect new entities to be added in the first few uses of the appliances.

# Configuration options
These options are supported in configuration.yaml:
```
home_connect_alt:
  client_id: < Your Client ID >
  client_secret: < You Client Secret >
  name_template: $brand $appliance - $name
  language: < Supported langage code >  
  sensor_value_translation: <server | local>  
  api_host: <Home Connect API host name (only use for China)>
```
## Parameters:
* *client_id* (required) - The Client ID of your Home Connect app.
* *client_secret* (required) - The Client Secret of your Home Connect app.
* *name_template* (optional - default = "$brand $appliance - $name") - 
  Defines the template used for rendering entity names. The following placeholders are supported and will be replaced dynamically when rendering the name:  
  $brand - The brand name of the appliance ("Bosch", "Siemens", etc.)  
  $appliance - The type of the appliance ("Washing machine", "Dishwasher", etc.)  
  $name - The name of the entity
* *language* (optional - default = "en") - 
  Indicates the language to use for entity names. The translation is automatically loaded from the Home Connect service and must  be one of its [supported languages](https://api-docs.home-connect.com/general?#supported-languages).
* *sensor_value_translation* (optional - default = "local") - Indicates how sensor values shhould be translated to friendly names.  
  When set to **"local"** (the default) the integration will use the raw ENUM values documented in the Home Connect documentation for sensors with string values. In that case the integration relies on the Home Assistant translation mechanism and translation files to translate these values into friendly names. The benefit of this approach is that sensor values used by the integration are language independent and match the values documented in the Home Connect API.  
  When set to **"server"** sensor values are translated to friendly names using the Home Connect service. In this mode the internal values of string sensors will be translated and the translated values must be used in scripts referring to those sensors.  

  **_Note:_** Select box values are always translated localy so they require the trsnalation files to contain all the possible values.
* *api_host* (optional) - If you are based in China, and only then, set this to *https://api.home-connect.cn*  

After the integration is configured READ THE FAQ then add it from the Home-Assistant UI.  

</br>

# FAQ
* **I get errors on the browser window that pops-up after installing the integration for the first time**
  This popup window is trying to log you into the Home Connect website to establish a connection with the integration. If you get an error at this stage it means you didn't follow the setup instuctions carefully enough, so make sure that you do.  
  Also make sure that you open https://my.home-assistant.io/ and configure the URL of your Home-Assistant server.  
  **NOTE:** If you are modifying an existing Home-Connect App then it may take up to 2 hours for the changes to take effect, so make sure you wait long enough. 
* **Some of my appliances are not showing up after I added the integration**  
  This is most commonly caused by two reasons:
  1. The appliance must be powered on and connected to the Home Connect service to be discovered. Once the missing devices are turned on and connected, they will automatically be discovered and added by the integration.  
  This can be verified in the Home Connect mobile app **but only while the wifi on the phone is turned OFF**. If the devices are active in the mobile app while the phone's wifi is turned off then please open an issue with a debug log to report it.
  2. Due to some unreasonable rate limits set by BSH, there is a limit of about 3 
  appliances loaded per minute. If you have more, expect the initial load to take longer. The integration will wait for the service to become available and continue loading the rest of the appliances. You may have to refresh your screen to see them in Home Assistant after they were added.

* **I've restarted Home Assistant a few times and now all my appliances are unavilable**  
  This is, again, related to the Home Connect rate limits. Every time you restart Home Assistant the integration makes a few API calls to the service and if that happens too often it may block for up to 24 hours. The best way to fix this is to wait a day and restart Home Assistant again.

* **I select a program or option but nothing happens on the appliance**  
  Make sure the appliance is turned on. Typically the integration will automatically detect appliances that are turned off or disconnected from the network and disable them in Home Assistant but it may happen that it fails to detect that and then attempting make any changes to setting will fail.

* **I try to start the selected program by pressing the button but nothing happens**  
  Make sure Remote Start is enabled on the appliance

* **Sensor related to program progress are showing up as unavailable**  
  These sensors only become available when the appliance is running a program.

* **All the program, option and settings selection boxes, switches and numbers are disabled**  
  Make sure that *Remote Control* is allowed on the appliance. It is typically enabled by default but gets temporarily disabled when changing setting on the appliance itself.

* **The *Start* button is disabled**  
  Make sure that *Remote Control Start* is allowed on the appliance, it's disabled by default. 

* **Sensor values or select boxes have values like BSH.Common.EnumType.PowerState.Off**
  Start by reading where these values are coming from in the explanation of the *sensor_value_translation* config param above. You can also set that parameter as a workaround for missing translations in sensor values. 
  Regardless if you decide to use the language parameter or not it would be great if you can report these values so they can be properly translated. Ideally, if you know how, please update the translation files directly (at least the English ones) and create a PR. If you don't know how then you can add them to issue [#26](https://github.com/ekutner/home-connect-hass/issues/26) and I will add them to the translation files.  

  **_Note:_** It is expected to see these values in sensor history data. This is currently a limitation/bug in Home Assistant which doesn't translate these values.

</br>

# Known Issues
* If you have more than 3 connected devices (or you just play a lot with settings), you may hit the daily API rate limit set by the Home Connect API. The limit is set to 1000 calls per day and when it is hit the API is blocked for 24h. During that time, the integration will not get updated with new states and you won't be able to select and options or start a program.  
If you hit this limit frequently, please open an issue with a debug log. I'll try to see if there is a way to reduce some calls. However, as of now, there is nothing I can do about it and the Home Connect team was unwilling to increase this limit, which hurts their best customers so there is nothing I can do about it. 

# Reporting issues / bugs
This integration is in early beta and not stable yet. If you are going to be testing this integration please add this to your configuration.yaml file then restart HA:
```
logger:
  default: warn
  logs:
    home_connect_async: debug
    home_connect_alt: debug
    custom_components.home_connect_alt: debug
```

When reporting an issue press the **Home Connect Debug** button and attach the HA log file to your issue report.

</br>

# Beta Releases
Major changes will be released in beta versions to reduce the impact on existing users. It is strongly advised not to install the beta version unless you are a part of an issue that specifically calls for installing one and when you are, you should only install the specified version (as there may be several issues handled in separate beta versions at the same time).  
To install a beta version go to the HACS dashboard in the Home Assistant UI:
* Click on Integrations locat the "Home Connect Alt" integration. 
* Click on the three dot menu for the integration and select "Redownload". 
* In the popup dialog make sure "Show beta versions" is enabled and then select the version you need to install.

<br>

# Automation Notes
* There is a special sensor called **Home Connect Status** which shows the status of the integration. It has the following values:
  * INIT - The integration is initializing
  * RUNNING - The integration has started running 
  * LOADED - The integration finished loading the initial data from the Home Connect service
  * READY - The integration has successfully subscribed to real time updates from the cloud service and is now fully functional. 
  * BLOCKED - The Home Connect service has blockde additional API calls for a period of time due to exceeding the service rate limits 
  *It may take up to one minute to go from LOADED to READY*

* The integration exposes the events fired by the service as Home Assistant events under the name: **"home_connect_alt_event"**
* The integration exposes two triggers for easy automation:
  * select_program
  * start_program 

# Legal Notice
This integration is not built, maintained, provided or associated with BSH in any way.
