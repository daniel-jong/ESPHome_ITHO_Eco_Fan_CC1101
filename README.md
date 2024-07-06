# ESPHome ITHO CVE ECO-FAN 2 control
Library for NodeMCU ESP8266 in combination with Hassio Home Assistant ESPHome ITHO Eco Fan CC1101
Code is optimized for Itho CVE Eco after 2011.


Trying to get ESPHome to mimic what is comprised in
 
 - https://github.com/jodur/ESPEASY_Plugin_ITHO/blob/master/_P145_Itho.ino
 - https://github.com/adri/IthoEcoFanRFT / https://github.com/supersjimmie/IthoEcoFanRFT


## Wiring schema used:

```
Connections between the CC1101 and the ESP8266 or Arduino:
CC11xx pins    ESP pins Arduino pins  Description
*  1 - VCC        VCC      VCC           3v3
*  2 - GND        GND      GND           Ground
*  3 - MOSI       13=D7    Pin 11        Data input to CC11xx
*  4 - SCK        14=D5    Pin 13        Clock pin
*  5 - MISO/GDO1  12=D6    Pin 12        Data output from CC11xx / serial clock from CC11xx
*  6 - GDO2       04=D1    Pin  2        Programmable output
*  7 - GDO0       ?        Pin  ?        Programmable output (NOT CONNECTED)
*  8 - CSN        15=D8    Pin 10        Chip select / (SPI_SS)
```


### Software used
Install the ESPHome addon for Home Assistant. I like to use the program ESPHome flasher on my laptop for flashing the firmware on my NodeMCU.

## Prepairing your Home Assistant for the ITHO controller
Open your `configuration.yaml` file and insert the following lines of code: (I like to put this code into fans.yaml and insert `fan: !include fans.yaml` in my configuration.yaml file)
```
fan:
  - platform: template
    fans:
      mechanical_ventilation:
        friendly_name: "Mechanische afzuiging"
        value_template: >
          {{ "off" if states('sensor.fan_speed') == 'off' else "on" }}
        percentage_template: >
          {% set speed = {'off': 0, 'low': 25, 'medium': 50, 'high': 75, 'full': 100} %}
          {{ speed [states('sensor.fan_speed')] | int }}
        turn_on:
          service: switch.turn_on
          data:
            entity_id: switch.fan_high
        turn_off:
          service: switch.turn_on
          data:
            entity_id: switch.fan_standby
        set_percentage:
          service: switch.turn_on
          data_template:
            entity_id: >
              {% set id_mapp = {0: 'switch.fan_standby', 25:'switch.fan_low', 50:'switch.fan_medium', 75:'switch.fan_high', 100:'switch.fan_full'} %}
              {{id_mapp[percentage]}}
        speed_count: 4
```

## ESPHome Configuration
I created a new device in Home Assistant ESPHome addon (named itho_eco_fan), and choose platform "ESP8266" and board "nodemcuv2". After that, I changed the YAML of that device to look like this: 

**DON'T COMPILE THE SOURCE YET!** Just save the YAML config and continue!

```
esphome:
  name: itho_eco_fan
  platform: ESP8266
  board: nodemcuv2
  includes: 
    - itho_eco_fan/itho/cc1101.h
  libraries:
    - SPI
    - Ticker
    - https://github.com/daniel-jong/ESPHome_ITHO_Eco_Fan_CC1101.git
    
  #Set ID from remotes that are used, so you can identify the root of the last State change
  on_boot:
    then:
      - lambda: |-
          Idlist[0]={"89,209,0,"Badkamer"};
          Idlist[1]={"89,209,0,"Keuken"};
          Idlist[2]={"ID3","ID3"};
         # Mydeviceid="HomeAssistant";
         # id(swfan_low).turn_on(); //This ensures fan is at low-speed at boot

wifi:
  ssid: "WiFi SSID"
  password: "wifi_password"

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "Itho Eco Fan Fallback Hotspot"
    password: "IEdsgfeESFEzS"

captive_portal:

# Enable logging
logger:
  level: verbose # Enable this line to find out the ID of your remote.

# Enable Home Assistant API
api:
  password: "api_password"

ota:
  password: "ota_password#"
  
switch:
- platform: custom
  lambda: |-
    auto fansendstandby = new FanSendStandby();
    App.register_component(fansendstandby);
    return {fansendstandby};
  switches:
    name: "FanSendStandby"
    id: swfan_standby
    icon: mdi:fan
    
- platform: custom
  lambda: |-
    auto fansendlow = new FanSendLow();
    App.register_component(fansendlow);
    return {fansendlow};
  switches:
    name: "FanSendLow"
    id: swfan_low
    icon: mdi:fan

- platform: custom
  lambda: |-
    auto fansendmedium = new FanSendMedium();
    App.register_component(fansendmedium);
    return {fansendmedium};
  switches:
    name: "FanSendMedium"
    id: swfan_medium
    icon: mdi:fan

- platform: custom
  lambda: |-
    auto fansendhigh = new FanSendHigh();
    App.register_component(fansendhigh);
    return {fansendhigh};
  switches:
    name: "FanSendHigh"
    id: swfan_high
    icon: mdi:fan

- platform: custom
  lambda: |-
    auto fansendfull = new FanSendFull();
    App.register_component(fansendfull);
    return {fansendfull};
  switches:
    name: "FanSendFull"
    id: swfan_full
    icon: mdi:fan

- platform: custom
  lambda: |-
    auto fansendt1 = new FanSendIthoTimer1();
    App.register_component(fansendt1);
    return {fansendt1};
  switches:
    name: "FanSendTimer1"

- platform: custom
  lambda: |-
    auto fansendt2 = new FanSendIthoTimer2();
    App.register_component(fansendt2);
    return {fansendt2};
  switches:
    name: "FanSendTimer2"

- platform: custom
  lambda: |-
    auto fansendt3 = new FanSendIthoTimer3();
    App.register_component(fansendt3);
    return {fansendt3};
  switches:
    name: "FanSendTimer3"

- platform: custom
  lambda: |-
    auto fansendjoin = new FanSendIthoJoin();
    App.register_component(fansendjoin);
    return {fansendjoin};
  switches:
    name: "FanSendJoin"

text_sensor:
- platform: custom
  lambda: |-
    auto fanrecv = new FanRecv();
    App.register_component(fanrecv);
    return {fanrecv->fanspeed,fanrecv->fantimer,fanrecv->Lastid};
  text_sensors:
    - name: "FanSpeed"
      icon: "mdi:transfer"  
    - name: "FanTimer"
      icon: "mdi:timer"
    - name: "fanLastID"
      icon: "mdi:id-card"
```

## Add the cc1101.h file (interface class to include/use the CC1101 library)
You're almost done! With the "Home Assistant Configurator" I navigate to the folder `esphome/itho_eco_fan`. Create the folder `itho` and go into the folder. Create the file `cc1101.h` and add the following contents to the file:

```
#include "esphome.h"
#include "IthoCC1101.h"
#include "Ticker.h"

// List of States:
// 0 - Itho ventilation unit to standby
// 1 - Itho ventilation unit to lowest speed
// 2 - Itho ventilation unit to medium speed
// 3 - Itho ventilation unit to high speed
// 4 - Itho ventilation unit to full speed
// 13 -Itho to high speed with hardware timer (10 min)
// 23 -Itho to high speed with hardware timer (20 min)
// 33 -Itho to high speed with hardware timer (30 min)

typedef struct { String Id; String Roomname; } IdDict;

// Global struct to store Names, should be changed in boot call,to set user specific
IdDict Idlist[] = { {"ID1", "Controller Room1"},
					{"ID2",	"Controller Room2"},
					{"ID3",	"Controller Room3"}
				};

IthoCC1101 rf;
IRAM_ATTR void ITHOinterrupt(); // ICACHE_RAM_ATTR;
void ITHOcheck();

// extra for interrupt handling
bool ITHOhasPacket = false;
Ticker ITHOticker;
int State=1; // after startup it is assumed that the fan is running low
int OldState=1;
int Timer=0;

String LastID;
String OldLastID;
String Mydeviceid = "ESPHOME"; // should be changed in boot call,to set user specific

long LastPublish=0; 
bool InitRunned = false;

// Timer values for hardware timer in Fan
#define Time1      10*60
#define Time2      20*60
#define Time3      30*60

TextSensor *InsReffanspeed; // Used for referencing outside FanRecv Class

String TextSensorfromState(int currentState)
{
	switch (currentState)
	{
	    case 0:
	        return "Standby";
    	case 1: 
    		return "Low";
    		break;
    	case 2:
    		return "Medium";
    		break;
    	case 3: 
    		return "High";
    		break;
    	case 13: case 23: case 33:
    		return "High(T)";
    	case 4: 
    		return "Full";
    		break;
        default:
            return "Unknown";
    	}
}

class FanRecv : public PollingComponent {
  public:

    // Publish 3 sensors
    // The state of the fan, Timer value and Last controller that issued the current state
    TextSensor *fanspeed = new TextSensor();
    // Timer left (though this is indicative) when pressing the timer button once, twice or three times
    TextSensor *fantimer = new TextSensor();
	// Last id that has issued the current state
	TextSensor *Lastid = new TextSensor();

    // For now poll every 1 second (Update timer 1 second)
    FanRecv() : PollingComponent(1000) { }

    void setup() {
      InsReffanspeed = this->fanspeed; // Make textsensor outside class available, so it can be used in Interrupt Service Routine
      
      ESP_LOGD("custom", "Startup");
      
      // Test RF chip
      uint8_t chipVersion = rf.getChipVersion();
      if (chipVersion)
      {
         ESP_LOGD("custom", "CC1101 RF module found");
      }
         else
      {
         ESP_LOGD("custom", "NO RF module found");
      }

      ESP_LOGD("custom", "rf init");
      rf.init();

      rf.setDefaultID(150, 149, 129);
      rf.setBindAllowed(false);
      rf.setAllowAll(true); // This disables filtering of recieved events alowing you to see device ids
     
      rf.addRFDevice(150, 0, 0, RemoteTypes::RFTCVE, true);
      rf.addRFDevice(150, 0, 0, RemoteTypes::RFTCO2, true);
      rf.addRFDevice(89, 0, 0, RemoteTypes::RFTCO2, true);
      rf.addRFDevice(6, 0, 0, RemoteTypes::RFTAUTO, false);
      ESP_LOGD("custom", "rf post init");

      // Followin wiring schema, change PIN if you wire differently
      pinMode(D2, INPUT);
      //pinMode(D1, INPUT);
      attachInterrupt(D2, ITHOinterrupt, RISING);      
      //attachInterrupt(D1, ITHOcheck, RISING);
			
      InitRunned = true;
    }

    void update() override {
        if (State >= 10)
		{
			Timer--;
		}

		if ((State >= 10) && (Timer <= 0))
		{
			State = 1;
			Timer = 0;
			fantimer->publish_state(String(Timer).c_str()); // this ensures that times value 0 is published when elapsed
		}
		//Publish new data when vars are changed or timer is running
		if ((OldState != State) || (Timer > 0)|| InitRunned)
		{
			fanspeed->publish_state(TextSensorfromState(State).c_str());
			fantimer->publish_state(String(Timer).c_str());
			Lastid->publish_state(LastID.c_str());
			OldState = State;
			InitRunned = false;
		}
    }


};

// Figure out how to do multiple switches instead of duplicating them
// we need
// send: standby, low, medium, high, full
//       timer 1 (10 minutes), 2 (20), 3 (30)
// To optimize testing, reset published state immediately so you can retrigger (i.e. momentarily button press)

class FanSendFull : public Component, public Switch {
  public:
    void write_state(bool state) override {
      if ( state ) {
        rf.sendCommand(IthoFull);
        State = 4;
		Timer = 0;
		LastID = Mydeviceid;
        publish_state(!state);
      }
    }
};

class FanSendHigh : public Component, public Switch {
  public:

    void write_state(bool state) override {
      if ( state ) {
        rf.sendCommand(IthoHigh);
        State = 3;
		Timer = 0;
		LastID = Mydeviceid;
        publish_state(!state);
      }
    }
};

class FanSendMedium : public Component, public Switch {
  public:

    void write_state(bool state) override {
      if ( state ) {
        rf.sendCommand(IthoMedium);
        State = 2;
		Timer = 0;
		LastID = Mydeviceid;
        publish_state(!state);
      }
    }
};

class FanSendLow : public Component, public Switch {
  public:

    void write_state(bool state) override {
      if ( state ) {
        rf.sendCommand(IthoLow);
        State = 1;
		Timer = 0;
		LastID = Mydeviceid;
        publish_state(!state);
      }
    }
};

class FanSendStandby : public Component, public Switch {
  public:

    void write_state(bool state) override {
      if ( state ) {
        rf.sendCommand(IthoAway);
        State = 0;
		Timer = 0;
		LastID = Mydeviceid;
        publish_state(!state);
      }
    }
};

class FanSendIthoTimer1 : public Component, public Switch {
  public:

    void write_state(bool state) override {
      if ( state ) {
        rf.sendCommand(IthoTimer1);
        State = 13;
		Timer = Time1;
		LastID = Mydeviceid;
        publish_state(!state);
      }
    }
};

class FanSendIthoTimer2 : public Component, public Switch {
  public:

    void write_state(bool state) override {
      if ( state ) {
        rf.sendCommand(IthoTimer2);
        State = 23;
		Timer = Time2;
		LastID = Mydeviceid;
        publish_state(!state);
      }
    }
};

class FanSendIthoTimer3 : public Component, public Switch {
  public:

    void write_state(bool state) override {
      if ( state ) {
        rf.sendCommand(IthoTimer3);
        State = 33;
		Timer = Time3;
		LastID = Mydeviceid;
        publish_state(!state);
      }
    }
};

class FanSendIthoJoin : public Component, public Switch {
  public:

    void write_state(bool state) override {
      if ( state ) {
        rf.sendCommand(IthoJoin);
        State = 1111;
        Timer = 0;
        publish_state(!state);
      }
    }
};

IRAM_ATTR void ITHOinterrupt() {
	ITHOticker.once_ms(10, ITHOcheck);
}

int RFRemoteIndex(String rfremoteid)
{
	if (rfremoteid == Idlist[0].Id) return 0;
	else if (rfremoteid == Idlist[1].Id) return 1;
	else if (rfremoteid == Idlist[2].Id) return 2;
	else return -1;
}

void ITHOcheck() {
  noInterrupts();
  
  if (rf.receivePacket()) {
	//ESP_LOGD("itho_rf", "We got a new packet over here");
    IthoPacket *packet = rf.checkForNewPacket();
    rf.parseMessage(packet);
    
	uint8_t *lastID = rf.getLastID(packet);
    IthoCommand cmd = rf.getLastCommand(packet);
	RemoteTypes remtype = rf.getLastRemType(packet);
    
	//String Id = rf.getLastIDstr(packet, false);
	String Id = String(lastID[1]) + "," + String(lastID[2]) + "," + String(lastID[3]);
	
	int index = RFRemoteIndex(Id);
    
    if (index >= 0) { // Only accept commands of controls that are in the list
        switch (cmd) {
          case IthoUnknown:
            ESP_LOGD("itho_rf", "Unknown command");
            break;
          case IthoAway:
            ESP_LOGD("itho_rf", "IthoAway");
            State = 0;
            Timer = 0;
            LastID = Idlist[index].Roomname;
          case IthoLow:
            ESP_LOGD("itho_rf", "Speed to Low");
            State = 1;
            Timer = 0;
            LastID = Idlist[index].Roomname;
            break;
          case IthoMedium:
            ESP_LOGD("itho_rf", "Medium");
            State = 2;
            Timer = 0;
            LastID = Idlist[index].Roomname;
            break;
          case IthoHigh:
            ESP_LOGD("itho_rf", "High");
            State = 3;
            Timer = 0;
            LastID = Idlist[index].Roomname;
            break;
          case IthoFull:
            ESP_LOGD("itho_rf", "Full");
            State = 4;
            Timer = 0;
            LastID = Idlist[index].Roomname;
            break;
          case IthoTimer1:
            ESP_LOGD("itho_rf", "Timer1");
            State = 13;
            Timer = Time1;
            LastID = Idlist[index].Roomname;
            break;
          case IthoTimer2:
            ESP_LOGD("itho_rf", "Timer2");
            State = 23;
            Timer = Time2;
            LastID = Idlist[index].Roomname;
            break;
          case IthoTimer3:
            ESP_LOGD("itho_rf", "Timer3");
            State = 33;
            Timer = Time3;
            LastID = Idlist[index].Roomname;
            break;
		  case IthoJoin:
	      case IthoLeave:
          case IthoAuto:
          case IthoAutoNight:
          case IthoCook30:
          case IthoCook60:
          case IthoTimerUser:
          case IthoJoinReply:
          case IthoPIRmotionOn:
          case IthoPIRmotionOff:
          case Itho31D9:
          case Itho31DA:
          case IthoDeviceInfo:
			ESP_LOGV("itho_rf", "Did not handle command %s", Id.c_str());
			break;
        }
    }
    else {
        ESP_LOGV("itho_rf","Ignored device-id: %s", Id.c_str());
		//Remtype is only sent when JOINing
		//ESP_LOGV("itho_rf","Ignored device-remotetype: %s", String(remtype));
    }
  }
  interrupts();
}
```
Save the file and go back to your device in the Home Assistant ESPHome addon.

## Flashing the NodeMCU and finishing the setup/installation
  1. "VALIDATE" the changes we made by clicking on the "VALIDATE" button/link. If everything is correct, Compile the code and download the BIN file.
  2. Restart Home Assistant to apply the configuration changes you made earlier.
  3. Attach the NodeMCU device with an USB cable to your laptop and start "ESPHome Flasher" with administrator privileges.
  4. Select the correct COM-port and select the BIN file you just created and downloaded.
  5. Flash the device.
  6. If you done everything right, your device should be up and running and connected to your WiFi network. In Home Assistant, navigate to "Settings > Integrations", your device should be found by Home Assistant. Click "Configure" and couple the device to Home Assistant (It will ask for the password you choose for the "API section" in the ESPHome device YAML).
  
Congrats! Your Itho Eco Fan Controller is connected to your Home Assistant, but we need to pair the device with your Itho Eco Fan.

## Pairing with Itho CVE Eco-fan 2
  1. Disconnect the Itho Eco Fan from your power net and wait +/- 30 seconds (This is what I did).
  2. Meanwhile, navigate in Home Assistant to "Settings > Integrations > Itho_eco_fan", you will see a couple of switches. Don't click any yet.
  3. Connect the Itho Eco Fan to your power net.
  4. Within +/- 20 seconds (as soon as you can), click on the switch behind "FanSendJoin".
  
If everything went right, which never happens the first 30 times, then your device should be connected to your Itho Eco Fan. Sometimes the fan goes into mode "2/Medium" when pairing, to let you know the pairing process went right.

## Have fun! Having problems?
https://gathering.tweakers.net/forum/list_messages/1690945
