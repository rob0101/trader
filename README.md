# Requirements: 
- you need to be an Amber Electric customer
- use Home Assistant
- have sensors for your own Amber buy and sell prices
- have a Sigenergy home battery system with modbus enabled
- have the TypQxQ "Sigenergy-Local-Modbus" integration installed
- run the Sig system in Remote EMS mode, i.e. with Smartshift already permanently disabled.
- be confident to have a go and assume all the risks of installing the files provided

This collection of files provides tools to help run automated trading of electricity, e.g. sell from the battery at specific rates when price thresholds are crossed.

One of the options will be battery -> grid selling for tiered prices, so you can export quickly to low SOCs for high spike prices and export slower to higher SOCs for lower prices. There will be a price trigger for charging from the grid, and price trigger for using the grid instead of the battery, a price trigger for exporting solar to the grid and more.

## Installation

### Install trader.yaml in a "packages" directory 

The trader.yaml file contains all the helpers needed to configure how you want to buy/sell electricity with Amber.

trader.yaml should be installed in a packages directory in your HA system.

Edit trader.yaml to tell it your Amber sensors for buy and sell prices.

```
template:
  - sensor:
      - name: "Max Grid Exports"
        state: 30
      - name: "My Amber Buy Price Sensor"
        state: "sensor.amber_true_five_min_price_mqtt"
        attributes:
          multiplier: "1"
      - name: "My Amber Sell Price Sensor"
        state: "sensor.amber_true_five_minute_export_price"  
        attributes:
          multiplier: "1"
```

You can use sensors that are in AUD by changing the multiplier from 1 to 100.
We deal in cents not dollar from here on.

### Tell HA to include the packages contents

Edit configuration.yaml and add (if not already there) an instruction to read packages from a specific directory.

```
# Use packages in /packages folder
homeassistant:
  packages: !include_dir_named packages
```

You need to create the directory called packages at the same level as configuration.yaml

Inside the "packages" directory, create a new file names "trader.yaml", edit it and paste in the contents of "trader.yaml".
If you have scp configured you can copy the file into place instead. (I find I need "scp -O" to send to my HA system).

It might be worth restarting HA to see if anything has broken so far, if all's well continue.

### Install a new dashboard

Next create a new dashboard view, edit the raw yaml and paste in the contents of the file config_dashboard.yaml REPLACING THE ORIGINAL CONTENTS.
Save the new dashboard.

![Screenshot 2025-08-23 at 2 51 24 pm](https://github.com/user-attachments/assets/2d797185-425e-4603-a261-b80f72c8d2b4)
![Screenshot 2025-08-23 at 2 52 44 pm](https://github.com/user-attachments/assets/b7cf41d4-b311-4b1c-8f9e-3b484e80c526)
![Screenshot 2025-08-23 at 2 53 41 pm](https://github.com/user-attachments/assets/adc874a9-a489-4ada-8888-277e42f32807)


You should then have something that looks similar to this

![Screenshot 2025-08-23 at 2 21 54 pm](https://github.com/user-attachments/assets/17e99af5-a259-4335-8d7e-b26d53b32cac)

At this stage, I only need testers to get this far and report issues.

I have some automations that will be uploaded once we have proven we can get this far.
