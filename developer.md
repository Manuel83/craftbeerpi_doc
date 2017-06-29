---

layout: blank
title: Extend CraftBeerPi
permalink: /developer/
---


## How to develop a new plugin

All plugins are in the folder `/home/pi/craftbeerp3/modules/plugins`.
Each folder is an own plugin.

If you like to create a new plugin just create a new folder under `/home/pi/craftbeerp3/modules/plugins`
Inside this folder create an new file called `__init__.py`. This is the entry point for your code.
Inside of this file place your code.

Folder Structure

```
+-- craftbeerp3 
|   +-- modules
|         +-- plugins
|                +-- YourCustomPlugin <- Just create a new folder 
|                       +-- __init__.py <- Your code goes here
|         +-- ... 
+-- ....
```

You can extend the following modules. 

* Actors
* Sensors
* KettleController (Logics to control the Kettle Temperature)
* FermentationController (Logics to control the Fermentation Temperature)
* Steps 

## Publish Plugin
1. Upload your __init__.py to your own GitHub repository. 
2. Create a README.md in your GitHub repository with a documentation for your plugin
3. To publish your plugin write an email to manuel@craftbeerpi.com containing the following information (Your name, GitHub URL of your Plugin)

## Custom Actor

```
from modules import cbpi
from modules.core.props import Property
from modules.core.hardware import ActorBase

@cbpi.actor
class SampleActor(ActorBase):
    #custom property 
    prop1 = Property.Text("Property1", True, "1")
    
    def on(self, power=0):
        '''
        Code to switch on the actor
        :param power: int value between 0 - 100
        :return: 
        '''
        print "SWITCH ON %s" % (self.prop1)
        
    def off(self):
        '''
        Code to switch off the actor
        :return: 
        '''
        print "SWITCH OFF"

    def set_power(self, power):
        
        '''
        Optional: Set the power of your actor
        :param power: int value between 0 - 100
        :return: 
        '''
        pass
       
```

## Custom Sensor

CraftBeerPi distinguishes between Active and Passive Sensors

* Active Sensors: Has to handle it's own loop and push data to CraftBeerPi
* Passive Sensors: Are executed every 5 seconds by CraftBeerPi to pull new data

### Active Sensor

```
from modules import cbpi
from modules.core.hardware import  SensorActive
from modules.core.props import Property

@cbpi.sensor
class DummyTempSensor(SensorActive):

    temp = Property.Number("Temperature", configurable=True, default_value=5)

    def get_unit(self):
        '''
        :return: Unit of the sensor as string. Should not be longer than 3 characters
        '''
        return "°C" if self.get_config_parameter("unit", "C") == "C" else "°F"

    def stop(self):
        '''
        Stop the sensor. Is called when the sensor config is updated or the sensor is deleted
        :return: 
        '''
        pass

    def execute(self):
        '''
        Active sensor has to handle its own loop
        :return: 
        '''
        while self.is_running():
            self.data_received(self.temp)
            self.sleep(5)

    @classmethod
    def init_global(cls):
        '''
        Called one at the startup for all sensors
        :return: 
        '''

```

### Passive Sensor`

Get's invoked by CraftBeerPi core every 5 seconds`

COMMING SOON

## Custom Kettle Controller

With a controller is possible to control the kettle temperature. You could code hysteresis or PID controller or a special controller for pump systems. All you need is to implement one single Pyhton class

```
from modules import cbpi
from modules.core.controller import KettleController
from modules.core.props import Property


@cbpi.controller
class SampleController(KettleController):

    # Custom Properties
       
    # will crate a number field for the user interface
    p1 = Property.Number("My Number Label", True, 0)
    
    # will create a text field for the user interface
    text1 = Property.Text("My Label", True, "Hello World")
    
    # Will create a drop down for the web interface
    s1 = Property.Select("Select Property", [1,2,3] )

    def stop(self):
        '''
        Invoked when the automatic is stopped.
        Normally you switch off the actors and clean up everything
        :return: None
        '''
        super(KettleController, self).stop()
        pass


    def init(self):
        '''
        Invoked when the kettle automatic is switched on. 
        :return: 
        '''
        pass
    
    def run(self):
        '''
        Each controller is exectuted in its own thread. The run method is the entry point
        :return: 
        '''
        while self.is_running():
            # YOUR LOGIC GOES HERE
            
            # Access the properties
            self.text1
            self.p1
            self.s1
            
            # get current kettle temperature
            self.get_temp()
            
            # get current kettle target temperature 
            self.get_target_temp()
            
            # switch heater on
            self.heater_on(100)
            
            # switch heater off
            self.heater_off()
            
            # get sensor value. The method takes the sensor id as in value.
            self.get_sensor_value(1)
           
            # Make sure to add a sleep between each iteration. Use self.sleep
            
            self.sleep(1)
```

## Custom Brew Step 

```
from modules.core.props import Property, StepProperty
from modules.core.step import StepBase
from modules import cbpi

@cbpi.step
class MyMashStep(StepBase):
    '''
    Just put the decorator @cbpi.step on top of a method. The class name must be unique in the system
    '''
    # Properties
    temp = Property.Number("Temperature", configurable=True)
    kettle = StepProperty.Kettle("Kettle")
    timer = Property.Number("Timer in Minutes", configurable=True)

    def init(self):
        '''
        Initialize Step. This method is called once at the beginning of the step
        :return: 
        '''
        # set target tep
        self.set_target_temp(self.temp, self.kettle)

    @cbpi.action("Start Timer Now")
    def start(self):
        '''
        Custom Action which can be execute form the brewing dashboard.
        All method with decorator @cbpi.action("YOUR CUSTOM NAME") will be available in the user interface
        :return: 
        '''
        if self.is_timer_finished() is None:
            self.start_timer(int(self.timer) * 60)

    def reset(self):
        self.stop_timer()
        self.set_target_temp(self.temp, self.kettle)

    def finish(self):
        self.set_target_temp(0, self.kettle)

    def execute(self):
        '''
        This method is execute in an interval
        :return: 
        '''

        # Check if Target Temp is reached
        if self.get_kettle_temp(self.kettle) >= int(self.temp):
            # Check if Timer is Running
            if self.is_timer_finished() is None:
                self.start_timer(int(self.timer) * 60)

        # Check if timer finished and go to next step
        if self.is_timer_finished() == True:
            self.next()

```

### Properties 

The follwoing properties are supported

```
# Will create a text field for the step configuraiton
t1 = Property.Number("Temperature", configurable=True)

# Will create a number field for the step configuraiton
n1 = Property.Number("Timer in Minutes", configurable=True)

# Will create a dropdown field for the step configuraiton.
s1 = Property.Select("My Select Field", [1,2,3])

# Will create a kettle dropdwon field for the step configuraiton. 
kettle = StepProperty.Kettle("Kettle")

# Will create a sensor dropdwon field for the step configuraiton. 
sensor = StepProperty.Sensor("Sensor")

# Will create a actor dropdwon field for the step configuraiton. 
actor = StepProperty.Actor("Actor")

```


 
