<p align="center">
	<img height="300" src="https://s.alicdn.com/@sc01/kf/H55421308235b45568c523f2684731916J.png">
</p>



# Proscenic M7 Pro ( Home Assistant package)

Since I purchased the Proscenic M7 PRO my goal has been to integrate it into Home Assistant.
Unfortunately, there are no solutions for a direct integration, as only the previous models (e.g. model 790t) have been integrated.
Speaking with some developers I was suggested to create a local proxy, with which to intercept calls made by the application for smartphones, so I can replicate these calls and finally integrate the vacuum in the home automation system.

After a lot of work I was able to intercept such information.

I have noticed an increasing interest in this vacuum model, so I decided to speed up a little bit and start publishing what I did. 

Almost all the functions have been integrated, something is still missing. I hope to be able to publish as many things as possible in the shortest time possible.

All your help is precious, feel free to collaborate. Thanks


## Installation
Package configuration must be enabled. Just add to your *configuration.yaml*
```
homeassistant:
  packages: !include_dir_named packages
```
then download the folder *proscenic_m7_pro* and paste it into the packages folder.


*The following steps are taken from the immense work done by the user [Fluepke](https://github.com/Fluepke/proscenic), whom I would like to thank for his enormous work. Previously I got the token but with a more complex method, so thanks for this simplified procedure.*

From linux terminal run the following

```
curl -v -k -X POST -H "os: i" -H "Content-Type: application/json" -H "c: 338" -H "lan: en" -H "Host: mobile.proscenic.com.de:443" -H "User-Agent: ProscenicHome/1.7.8 (iPhone; iOS 14.2.1; Scale/3.00)" -H "v: 1.7.8" -d "{\"state\":\"欧洲\",\"countryCode\":\"49\",\"appVer\":\"1.7.8\",\"type\":\"2\",\"os\":\"IOS\",\"password\":\"$(echo -n $PASSWORD | md5sum)\",\"registrationId\":\"13165ffa4eb156ac484\",\"language\":\"EN\",\"username\":\"$LOGINUSER\",\"pwd\":\"$PASSWORD\"}" "https://mobile.proscenic.com.de/user/login"
```

Obviously it is necessary to replace *$LOGINUSER* and *$PASSWORD* with the relative access data to the Proscenic Home application.

You will get a response like this:
```
{
  "code": 0,
  "msg": "success",
  "data": {
    "token": "XXXXXXXXXXXXXXX",
    "uid": "XXXXXXXXXXXXXXX",
    "equipcount": 0,
    "nickname": null,
    "recieveMsg": false,
    "countryCode": "49",
    "homeId": null
  }
}
```
Then run
```
curl "https://mobile.proscenic.com.de/user/getEquips/$LOGINUSER"  -d "username=$LOGINUSER"
```
You will get a response like this:
```
"content": [
      {
        "name": "M7 Pro",
        "code": "M7_PRO",
        "typeName": "CleanRobot",
        "model": "811_LDS",
        "sn": "XXXXXXXXXXXXXXX",
        "deviceId": null,
        "status": true,
        "imgUrl": "http://mobile.proscenic.com.de/images/M7_PRO.png",
        "homeId": null,
        "shared": false,
        "jump": "M7_PRO",
        "ctrlversion": null,
        "enabled": true,
        "scMac": null,
        "scSV": null,
        "faqUrl": "https://www.proscenic.com/support/faq-f0460-l9999.html",
        "cloud": 0,
        "type": "CleanRobot"
      }
]
```
Now insert your username, the token (from the first *curl* command) and the sn (from the second *curl* command) you just obtained into the *secrets.yaml* file contained within the *proscenic_m7_pro* folder


## Available functions
When your restart Home Assistant you will be able to:
* through the vacuum entity:
  * start a normal cleaning process
  * pause the cleaning process (only pause, not continue. you need to start the cleaning process again)
  * send the vacuum to the charging base
  * set the fan speed to "quiet", "standard" or "powerful" 

* through the scripts:
  * start a deep cleaning
  * collect dust (additional dust bin needed)
  * continue the cleaning process
  * (obviously) all the functions available in the vacuum entity

## Lovelace manual card
Just create a new manual card and paste the following code
```
type: entities
entities:
  - entity: vacuum.proscenic_m7_pro
  - entity: script.proscenic_clean
  - entity: script.proscenic_deep_cleaning
  - entity: script.proscenic_pause
  - entity: script.proscenic_continue
  - entity: script.proscenic_charge
  - entity: script.proscenic_collect_dust
  - entity: script.proscenic_powermode_quiet
  - entity: script.proscenic_powermode_standard
  - entity: script.proscenic_powermode_powerful
show_header_toggle: false
title: Proscenic M7 PRO
```


## Upcoming implementations

### Already obtained (just need to report them here)
- generic infos (name, model, serial number and others)
- firmware informations
- avaibility
- last message(s) (cleaning completed, tank emptying in progress and others)
- zone cleaning function (based on automatic partition)
- multi-zone cleaning function (based on user defined zones)
- Y shaped mopping function
- set Silent (on/off)
- set Equipment light (on/off)
- auto dust collection (enable/disable)
- set volume


### Still to be achieved (project finished and I'll finally be happy!).
- add the "continue" to the vaccum entity (unfortunately the vacuum template entity only accepts the pause function (meaning the same command for a restore), but in our case the commands would be different)
- battery information
- size of the area currently being cleaned
- time of the cleaning operation currently in progresss
- consumables information ( usage percentage and need of replacement)


## Features that will not be implemented
- Silent Mode: the "Silent Mode" menu allows you to set the time slots when the robot is set to Silent mode. It does not make sense to perform the operation via home automation hub
- Appoint Clean: You need the map to indicate the exact cleanup point. It may have sense only if you can get the exact point (which is possible), at that point you can create a script for cleaning that exact point. It doesn't make much sense, if you decide to implement it, it will be given a very low priority
- Area Clean: Cleans up the square around where it is located. Doesn't make sense via home automation hub, will be left only via app
- Remote control
