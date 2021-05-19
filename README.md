# Q1 Laser Controls Labview

This repository contains the Labview API for controlling the Q1 Laser. The main goal is to wrap the basic commands as vi's so they can be easily integrated into Labview programs that use the Q1 laser.

# Structure of commands

The Q1 laser is controlled over the LAN via HTTP protocols. Section 11 "Programming" of the user manual (ppg. 45 - 51) contains the instructions on how to program the laser. The user manual is on the BvL Wiki on the [Q1 Laser page](https://tapajo.physics.mcgill.ca/neutrino/wiki/bnl/index.php/Q1_Laser) along with all the notes and documentation.

All commands are issued as JSON-formatted strings. Note that the JSON-formatted sting must be HTTP-encoded (i.e. '{' -> '%7B' etc.). 
```
http://172.16.0.47:7557/send?JSON_OBJECT
```
For ease of Labview implementation, this API will generally use the concatenate strings function to build the command string from `http://172.16.0.47/send?` and the JSON command and any inputs to the JSON command. This "normal" string is then converted to the HTTP-encoded version with `Escape HTTP URL.vi` that comes with Labview (Connectivity > Web Services > Conversion). This can then be sent to the laser via the `GET.vi` from Labview. 

It is also required to use `Open Handle.vi` (Data Communication > Protocols > HTTP) to enter the [credentials of the laser](https://tapajo.physics.mcgill.ca/neutrino/wiki/bnl/index.php/Q1_Laser) and store them for the remainder of the session. Since many operations will be done with the laser in a given vi that uses this repository, `LaserInitialize.vi` has been included to simply create the http client handle with the proper credentials hard-coded (these will not change for the laser). Each vi in this repository indended for use as a sub-vi should therefore have the top left and right terminals of the icon wired to be the http client in and out respectively. That means ther is **NO** use of either `Open Handle.vi` or `Close Handle.vi` in the sub-vi, this is done in the "master" vi written by users of this repository.

To check if your command was issued properly, you can use the [Web Control Panel](http://172.16.0.47:7557/) to see what the laser's current status and settings are! 

# Functions

## Send Command Example
This is a simple example vi to show how commands are issued. It is for demonstration and troubleshooting only, **do not embed this in other vi's!**  

### Inputs:  
**JSON Command:** The command to be issued as a "normal" string. Thyese can by copy/pasted from the manual.

### Outputs:  
**Headers:** Header of HTTP response from laser  
**Body:** Body of the HTTP response from the laser. Usually `{"message":"success"}` or `{"message":"failed"}`  

## Laser Initialize
This vi initializes the HTTP handle with credentials for the Q1 laser

### Inputs:  
**None**

### Outputs:  
**client handle out:** HTTP handle with credentials for Q1 laser communication 

## Check Current Status
This is a vi to show the current settings of the laser as an un-parsed JSON string.

### Inputs:  
**None**

### Outputs:  
**Headers:** Header of HTTP response from laser  
**Body:** Body of the HTTP response from the laser giving the current values for all the settings in JSON format.

## Laser Status
This is a vi to show a few parameters of the laser's current status

### Inputs:  
**client handle in:** HTTP handle with credentials for Q1 laser communication

### Outputs:  
**client handle out:** HTTP handle with credentials for Q1 laser communication  
**LaserOn?:** True if laser is running, False if laser is stopped  
**HU-Temp-ready:** True if harmonic unit temperature is ready for operation  
**laser-error-code:** error code from laser head  
**hu-error-code:** error code from harmonic unit  
**rep-rate:** current repetition rate of the laser in Hz  
**h1-transmission:** current transmission setting for the attenuator in %  
**LD1-set-current:** Laser diode set current in A  
**h1-zero-transmission-offset:** Attenuator offset for minimum transmission  

## Laser On/Off
This is a vi to turn on and off the laser

### Inputs:  
**client handle in:** HTTP handle with credentials for Q1 laser communication  
**Laser on/off:** Boolean [True: Laser on , False: Laser off]

### Outputs:  
**client handle out:** HTTP handle with credentials for Q1 laser communication  
**LaserOn?:** Boolean [True: Laser on , False: Laser off]  
 