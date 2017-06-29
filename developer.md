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

##Custom Actor Example

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

 
