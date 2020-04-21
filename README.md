# cabinet_lights
ESPHome / PWM Dimming Under Cabinet Lighting

# Overview
I've wanted to add under cabinet LED strip lighting to my kitchen for a while.  At the end of last year (2019) I gutted and remodeled out kitchen.  I didn't have the time, or foresight to plan ahead and run any wires or figure this out.  I came up with a plan.  I have two seperate parts of the cabinets, two "zones".  Each does have power avaialble, theres a plug for the microwave in the cabinet above the microwave, and a plug behind the fridge.  The low voltage wire (speaker wire) is small enough to run inside the cabinets and not really be seen.  It is pinched against the face of the cabinet on the inside, held in place by the shelves.  An ESP32 board drives a MOSFET for PWM dimming, all controlled via HomeAssistant.

https://youtu.be/GbK3mrdwiZg

# Hardware

![1](https://i.imgur.com/3zArk9U.png)

**ESP32** - I bought a two pack off of Amazon for this project.  Most of what i used is from Amazon.  MELIFE was the brand, went with those becuase they use an AMS1117 voltage regulator which can handle 12v.  Also, the reason for an ESP32 vs. an ESP8266 is the PWM output on the ESP32 is hardware based, and there are multiple PWM capable pins so you could use one board for multiple different zones.

**RFP30N60LE MOSFET** - N-Channel MOSFET to handle the PWM pulses.  These work at logic level votage to control the gate, no need for a level shifter.  The GPIO pins run at 3.3v, these specific MOSFETs will work at that voltage.

**12V Power Supply** - I bought a 2-pack of 12v 2A wall plugs.  The entire strip (16' / 5m) pulls 44w (3.6A).  The two sections I have are under half of that total and they will be powered individually so the 2A power supplies are plenty.

**12v LED Strips** - NOT ALL LED STRIPS ARE CREATED EQUAL.  Spend the money on a quality strip.  I've bought 16' strip 2-packs for less than half of the cost of one strip that i used for this project.  It really makes a difference.  The light quality is much better (90+ CRI), the construction of the strip is way better, and the self adhesive backing is actually sticky and works VERY well.  I went with a 3000K "warm" white, but they're on the cooler end of the warm spectrum in my opinion.  Brand i used are HitLights 2835, 600 LEDs, 3000K, 44W, CRI 90+.

**Breadboard and Other General Items** - I have extra breadboard, so that's what i use.  Maybe someday I'll design a PCB for a project, but for now, this works.  Theres a 10k ohm resistor in there, some screw terminals (life saver), jumper wires (solid core pre-bent), speaker wire, soldering iron and supplies, a multimeter...

# Board / Circuit Assembly

Here is a crappy hand drawn circuit diagram for the direct 12v version.  One board in my setup is using a buck converter (pictured above), the other is 12v direct to the VIN and GND pins on the ESP32.  TEST YOUR BOARD FIRST.  Not all ESP boards are created equal, trust me.  The reason I'm using this one is because it uses the AMS1117 voltage regulator.  The spec sheet for that says it can handle 12v.  My expericne with that regulator is that it will definately handle 12v, but it does get a little warm.  I ran a couple NodeMCU boards off 12v over the holidays for addressable LED controllers, they held up just fine. One batch of NodeMCUs i bought had a different regulator.  That one did not appreciate being fed 12v and blew up, you can see the burn marks on the breadboard in the pics on this writeup.

![Awesome Hand Drawn "Schematic"](https://i.imgur.com/eU5YqFX.png)

12v is fed to the ESP32 via VIN and GND pins.  I'm using GPIO19 on the ESP32, there are many PWM capable pins, that's just the one i chose.  The ouput from the pin is fed through a 10k ohm resistor, then to the GATE pin on the MOSFET, check the datasheet for the pinout if you're unsure.  The SOURCE pin on the MOSFET is fed GND from the 12v PSU.  The DRAIN pin on the MOSFET is the output, that goes to the terminal block for the GND to the LED strip.  

![12v Direct Board](https://i.imgur.com/Y6J6LuR.png)

# LEDs

I measured and cut to length the 3 strips I needed for the 3 sections.  Two of them run off one board, the 3rd off the other.  I soldered some speaker wire leads with enough length to get up to the controller.  I bent the soldered connections 90 degrees so when the strips are adhered to the underside of the cabinet the lead will go straight up into a hole that was drilled.

![LED Solder Connection](https://i.imgur.com/UfGgnUc.png)

The wires hide nicely inside the face of the cabinet frame, the shelves pinch the wires in place so they really aren't visible in the main cabinets.  You can see the controller in the cabinet above the microwave.

![5](https://i.imgur.com/E7ML5zl.png)

How you install the LEDs is up to you, here are a couple more installation pics.

![6](https://i.imgur.com/CZEmyPY.png)
![7](https://i.imgur.com/hCcQBKr.png)
![8](https://i.imgur.com/FBe5AZv.png)
![9](https://i.imgur.com/atEh0PL.png)
![10](https://i.imgur.com/2AB7alg.png)

# Software

The ESP32 is running ESPHome.  The code is here --> https://pastebin.com/KgZyQPDb  I won't dive into the details there, it's pretty straight forward.  Using the LED_C PWM output in ESPHome.

I use HomeAssistant for my home automations.  The lights show up in HA as a dimmable light.

![11](https://i.imgur.com/7RBtWdR.png)

I do have two controllers for the zones but I want them to work as one by default.  I created a light group in HA in my config.yaml and added the two entities (light.cabinet_1 and light.cabinet_2) then called it Cabinet Lights.  Yaml for that here --> https://pastebin.com/Pm9umXtU

# Controlling Via Dimmer Switch & NodeRed

I had an extra MartinJerry wifi dimmer wall swtich installed that wasn't being used for anything useful.  I re-wired some lights in my kitchen, this switch location is right next to the cabinets which is nice.  It was part of a 3-way switch setup that I didn't like so I installed the dimmer but didn't hook it up to the light, just powered it on so it is essentially a software swtich.  The swtich is flashed with ESPHome and I expose the physical buttons to HA.  Code for that switch is here --->  https://pastebin.com/iqPkpLJr  So now when the physical buttons are pressed on the dimmer (that isn't actually wired to a light) we can pick it up in HA (NodeRed) and do stuff.

I'm using NodeRed, the flow is here -->  https://pastebin.com/XwdejWdy
![12](https://i.imgur.com/xbQ8eBb.png)

When the up button is clicked, the brightness of the light group is increased 15%, when down is clicked, brightness is reduced 15%.  If the up button is pressed for more than 500ms the brightness is taken to 100%.  When the power button is pressed and the lights are on, they are turned off.  When the power button is pressed and the lights are off, they're brought up to 50%.  This is a great way to use a "dummy" switch / device to control other devices that are not physically linked.  There is zero physical control of the LED strips which does mean that if HA / NodeRed take a crap, I can't control these lights (for now).

# Automating

![13](https://i.imgur.com/hTwNEgC.png)

I haven't automated THESE lights yet, I need to find a good place to shoe-horn them into my existing spider web of kitchen lighting automation....  Hey, they work, and the wife likes them, what more can you ask for?  ;-)
