# Requirements: 
- you need to be an Amber Electric customer
- use Home Assistant
- have sensors for your own Amber buy and sell prices
- have a Sigenergy home battery system with modbus enabled
- have the TypQxQ "Sigenergy-Local-Modbus" integration installed
- run the Sig system in Remote EMS mode ***
- have the File Editor add-on
- have power-flow-card and card-mod installed (HACS)
- be confident to have a go and assume all the risks of installing the files provided

*** as of August 2025 if you change from VPP mode (Amber) to Remote EMS mode (Home Assistant) there's no easy way to switch back to VPP mode. Amber are working on a fix. In the meantime you have to ask Amber to re-enable VPP mode.

This collection of files provides tools to help run automated trading of electricity, e.g. sell from the battery at specific rates when price thresholds are crossed.

One of the options will be battery -> grid selling for tiered prices, so you can export quickly to low SOCs for high spike prices and export slower to higher SOCs for lower prices. There will be a price trigger for charging from the grid, and price trigger for using the grid instead of the battery, a price trigger for exporting solar to the grid and more.

The dashboard included in this package should be seen as a starting point for your own work.  You can keep it, add to it or rewrite it.  The key components to make things work are in the trader.yaml file which you'll not need to change (unless you really want to).

Similarly, automations shared (at a later date) in this package can be used as they are, adapted or discarded.

## Installation

### Install trader.yaml in a "packages" directory 

The trader.yaml file contains all the helpers needed to configure how you want to buy/sell electricity with Amber.

trader.yaml should be installed in a packages directory in your HA system.

Edit trader.yaml to tell it your Amber sensors for buy and sell prices.
Change the max grid exports from 30 to your maximum allowed grid export value in kW

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
We deal in cents not dollar from here on, much like the Amber app.

### Tell HA to include the packages contents

Edit configuration.yaml and add (if not already there) an instruction to read packages from a specific directory.
To edit the file you need HA to have the **File Editor** installed, via Settings -> Add Ons.

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

![Screenshot 2025-08-29 at 1 10 54 pm](https://github.com/user-attachments/assets/6d23ab79-3b5c-431b-b1b8-af3b05170177)


at the time of the screenshot, the import (buy) price in my region was negative, so automations shut down solar and started to run the house from the grid and top up the battery too.   With negative buy prices on Amber, we get a credit for the electricity we consume from the grid.

### The logic behind the A-E battery sell prices and rates

The battery discharge (sell) plans named A to E are optional. They are checked against the current sell price to see which (if any) plan matches.  They are checked A to E, in that order.  As soon as one plan matches, it wins.  With that in mind you'd set Plan A to have the highest price and E have the lowest.

In normal operations you might not want all Plans to be active, perhaps just 2 plans, one to catch a price that you're happy to sell into slower and one you'd want to sell into really fast.  e.g. you might want to always sell from your battery at a rate of 2kW if the sell price exceeds 30c.  If the price spikes to say $1 you're probably happier to sell faster, maybe at 30kW (on a 3 phase system, 3 x 10kW per phase).  You want your exports to stop at some point and that's defined by the minimal battery State Of Charge (SOC). Again you might be happy to sell down to 10% if you're earing $1/kwh, but only to 50% SOC if the price is 32c/kWh.

During price spikes with Amber people often try to manage their battery to last until the end of a spike.  The forecasts might suggest 3 hours of high prices.  It's on those occassions you might want to enable more price points (Plans) for different export rates and minimum SOCs.

On the spike nights you might also choose to buy from the grid at unusually high prices. The Sigenergy batteries can change from selling to buying really quickly, so we can "buy the dips", e.g.  the price might be $18/kWh for a while and suddenly drop to 70c for 5m.  You can buy at 70c to sell at $15 later.

With so many batteries being controlled, Amber's Smartshift can get very slow to respond to these rapid price swings. Using buy and sell price triggers on our Sig systems gives us super fast responses.

### The logic of buying from the grid instead of drawing from the battery

This option exists so that you can choose to preserve what's in the battery if the grid import price is below a threshold.  Some days I have certain household loads that I want to run that exceed the solar output at the time. In the Sig's Maximum Self Consumption mode, the battery will be used to cover the shortfall from solar. The power used from the battery might be better saved to use later in the day or might be saved to sell later.  In those circumstances, buying from the grid to cover the solar shortfall, at say 5c/kWh can be a good move.

### Exporting excess solar

Often with Amber, it makes sense to sell your solar early in the day and late in the day.  These typically are the highest prices for solar.  Selling early or late makes sense if the sell price now is higher than what you will pay or have paid to import from the grid. It also makes sense if there's going to be plenty of solar for your battery/house later in the day.

### Preserve the battery

Various automated system such as Amber's Smartshift offer "preserve" modes designed to keep the battery at the current level for a specified time. The dashboard provided here offers a more usful preserve that's split into two types, preserving at or below a state of charge (SOC) or at or above a SOC, or both together. These extend the usefulness of the Sig's built-in charge and discharge cut-off limits.

### Automations

I've included (or will include) some example automations.  Create a new automation, edit it in yaml and paste the example file.  You then customise that automation via the friendly UI to suit your own needs.

Here's an ape charts view of 5m interval imports (blue) and exports (yellow). Above the axis is a cost, below is a negative cost (earnings).
![Screenshot 2025-09-19 at 10 59 41 am](https://github.com/user-attachments/assets/8214283e-cd6d-4d0e-a9df-a9b6735ef230)

