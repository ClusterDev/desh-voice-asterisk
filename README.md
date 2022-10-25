# Setting up Desh Voice on Asterisk

This document will guide you through setting up Desh Voice on your Asterisk installation.

## Basics
To run this, you'll need to set up the following:
- Asterisk 18
- ACL configuration to allow TCP connection to `voice-modules.clusterdev.com` on `port 80`
- Speech recognition module setup (follow instructions below)
- Dialplan configuration to create speech recognition request and to handle responses (see example below)

## Setting up speech recognition module
Run these commands on the machine where Asterisk is running:
```
$ cd /usr/src/
$ git clone https://gitlab.com/clusterdev/desh-voice-asterisk

$ cd /usr/src/desh-voice-asterisk/
$ ./bootstrap

# Change the /usr/src/asterisk-18.current/ with the correct path to your Asterisk setup's source code
$ ./configure --with-asterisk=/usr/src/asterisk-18.current/ --prefix=/usr

$ make
$ make install

$ cp /usr/src/desh-voice-asterisk/conf/res_speech_vosk.conf /etc/asterisk/
```

Edit `modules.conf` to load necessary modules
```
load = res_speech.so
load = res_http_websocket.so
load = res_speech_vosk.so
```

## Dialplan configuration
Refer the following code for a sample dialplan that uses our district selection module

Make sure to choose the appropriate speech recognition module using `SpeechDeactivateGrammar`. For example, `MDLcolr`, `MDLdist` etc.

```
[district_incoming]
exten => 111,1,Answer

; Play "Welcome to ABCD Bank"
same => n,Playback("/home/ubuntu/dev/media/district/intro-hello")

; Connect to Desh Voice server
same => n(listen),SpeechCreate
same => n,SpeechDeactivateGrammar(MDLdist)

; Play "Which district are you calling from?" while listening in the background
; This will timeout 5 seconds after playback is complete
same => n,SpeechBackground("/home/ubuntu/dev/media/district/intro-question", 5)

; Check for for the speech response and run appropriate logic
; In this example, it says "Okay, we'll connect you to <district> branch"
same =>n(check),GotoIf($["${SPEECH_TEXT(0)}" = "kasaragod"]?kasaragod)
same =>n,GotoIf($["${SPEECH_TEXT(0)}" = "kannur"]?kannur)
same =>n,GotoIf($["${SPEECH_TEXT(0)}" = "wayanad"]?wayanad)
same =>n,GotoIf($["${SPEECH_TEXT(0)}" = "kozhikode"]?kozhikode)
same =>n,GotoIf($["${SPEECH_TEXT(0)}" = "malappuram"]?malappuram)
same =>n,GotoIf($["${SPEECH_TEXT(0)}" = "palakkad"]?palakkad)
same =>n,GotoIf($["${SPEECH_TEXT(0)}" = "thrissur"]?thrissur)
same =>n,GotoIf($["${SPEECH_TEXT(0)}" = "ernakulam"]?ernakulam)
same =>n,GotoIf($["${SPEECH_TEXT(0)}" = "idukki"]?idukki)
same =>n,GotoIf($["${SPEECH_TEXT(0)}" = "kottayam"]?kottayam)
same =>n,GotoIf($["${SPEECH_TEXT(0)}" = "alappuzha"]?alappuzha)
same =>n,GotoIf($["${SPEECH_TEXT(0)}" = "pathanamthitta"]?pathanamthitta)
same =>n,GotoIf($["${SPEECH_TEXT(0)}" = "kollam"]?kollam)
same =>n,GotoIf($["${SPEECH_TEXT(0)}" = "trivandrum"]?trivandrum)

; If we didn't get a match, retry
; To avoid infinite number of retries, this should be tweaked
; to redirect to another flow after X number of retries.
same =>n,Wait(2)
same =>n,goto(listen)

; Play appropriate recording for each result
same => n(kasaragod),Playback("/home/ubuntu/dev/media/district/kasaragod")
; Write relevant logic here for this result
; For example, connect this call to an agent's mobile
same=>n,goto(bye)

; Do the same for other results
same => n(kannur),Playback("/home/ubuntu/dev/media/district/kannur")
same=>n,goto(bye)
same => n(wayanad),Playback("/home/ubuntu/dev/media/district/wayanad")
same=>n,goto(bye)
same => n(kozhikode),Playback("/home/ubuntu/dev/media/district/kozhikode")
same=>n,goto(bye)
same => n(malappuram),Playback("/home/ubuntu/dev/media/district/malappuram")
same=>n,goto(bye)
same => n(palakkad),Playback("/home/ubuntu/dev/media/district/palakkad")
same=>n,goto(bye)
same => n(thrissur),Playback("/home/ubuntu/dev/media/district/thrissur")
same=>n,goto(bye)
same => n(ernakulam),Playback("/home/ubuntu/dev/media/district/ernakulam")
same=>n,goto(bye)
same => n(idukki),Playback("/home/ubuntu/dev/media/district/idukki")
same=>n,goto(bye)
same => n(kottayam),Playback("/home/ubuntu/dev/media/district/kottayam")
same=>n,goto(bye)
same => n(alappuzha),Playback("/home/ubuntu/dev/media/district/alappuzha")
same=>n,goto(bye)
same => n(pathanamthitta),Playback("/home/ubuntu/dev/media/district/pathanamthitta")
same=>n,goto(bye)
same => n(kollam),Playback("/home/ubuntu/dev/media/district/kollam")
same=>n,goto(bye)
same => n(trivandrum),Playback("/home/ubuntu/dev/media/district/trivandrum")
same=>n,goto(bye)

; End the call
same => n(bye),Hangup()

```

## License
This is a modified fork of [vosk-asterisk](https://github.com/alphacep/vosk-asterisk) and is licensed under GPL-2.0.
