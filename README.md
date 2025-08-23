# Installation

# Requirements: you need to be an Amber customer. Use Home Assistant, have sensors for your own Amber buy and sell prices, have a Sigenergy system with modbus enabled.

Tools to help run automated trading of electricity from a Sigenergy home battery system, using TypQxQ's integration for Sig + Home Assistant + modbus (https://github.com/TypQxQ/Sigenergy-Local-Modbus)

The trader.yaml file contains all the helpers needed to configure how you want to buy/sell electricity with Amber.

trader.yaml should be installed in a packages directory in your HA system.

Edit trader.yaml to tell it your Amber sensors for buy and sell prices.

```emplate:
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

Next create a new dashboard view, edit the raw yaml and paste in the contents of the file config_dashboard.yaml
Save the new dashboard.

You should then have something that looks similar to this

![Screenshot 2025-08-23 at 2 21 54 pm](https://github.com/user-attachments/assets/17e99af5-a259-4335-8d7e-b26d53b32cac)

At this stage, I only need testers to get this far and report issues.

I have some automations that will be uploaded once we have proven we can get this far.
