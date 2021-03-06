# omxsync
python module to sync playback of several omxplayer instances

[![Build Status](https://travis-ci.org/markkorput/pyOmxSync.svg?branch=master)](https://travis-ci.org/markkorput/pyOmxSync)
[![Code Climate](https://codeclimate.com/github/markkorput/pyOmxSync/badges/gpa.svg)](https://codeclimate.com/github/markkorput/pyOmxSync)

## Summary

A tiny python module that syncs omxplayers over the network using sockets and a master/slave configuration. The syncing logic was initialy inspired by the [omxplayer-sync](https://github.com/turingmachine/omxplayer-sync) implementation, but designed to be easily implementable in your python project (and not as a stand-alone runnable script). It uses [python-omxplayer-wrapper](https://github.com/willprice/python-omxplayer-wrapper) to interface with the OMXPlayer process.


## Install

First make sure willprice's [python-omxplayer-wrapper](https://github.com/willprice/python-omxplayer-wrapper) is installed. Go to [https://github.com/willprice/python-omxplayer-wrapper](https://github.com/willprice/python-omxplayer-wrapper) for instructions.

On the slave you _might_ need to change your raspi's configuration to allow incoming broadcast messages (syncing happens over a broadcast channel by default):
```shell
echo 0 | sudo tee /proc/sys/net/ipv4/icmp_echo_ignore_broadcasts
```

Finally install the omxsync package
```shell
pip install omxsync
```

## Usage - master

```python

from omxplayer import OMXPlayer
from omxsync import Broadcaster

player = OMXPlayer('path/to/video.mp4')
broadcaster = Broadcaster(player, {'verbose': True})
broadcaster.setup()
player.play()

while player.playback_status() != "Stopped":
	broadcaster.update()
```

## Usage - slave

```python

from omxplayer import OMXPlayer
from omxsync import Receiver

player = OMXPlayer('path/to/video.mp4')
receiver = Receiver(player, {'verbose': True})
receiver.setup()
player.play()

while player.playback_status() != "Stopped":
	receiver.update()
```

## Advanced Options

#### Use a custom network port (the broadcaster and receiver must use the same port)

```python
receiver = Receiver(player, {'port': 3001}) # default: 1666
broadcaster = Broadcaster(player, {'port': 3001}) # default: 1666
```

#### Use specific remote IP-adresses (instead of broadcasting). 

Using '0.0.0.0' for the receiver will accept both broadcasted data and data send specifically to the receiver. Using '255.255.255.255' on the broadcaster will broadcast the data Alternatively you could send to a specific machine by using the slave's IP address

```python
receiver = Receiver(player, {'host': '127.0.0.1'}) # default: '0.0.0.0'
broadcaster = Broadcaster(player, {'port': '192.168.2.5'}) # default: '255.255.255.255'
```

#### Specify a custom broadcast interval (in seconds)

```python
broadcaster = Broadcaster(player, {'interval': '5.0'}) # default: 1.0
```

### Custom slave-syncing parameters

Tolerance is the maximum amount of time that the slave is allowed to be ahead of behind on the master before syncing measures will be taken (specified in seconds).

```python
receiver = Receiver(player, {'tolerance': 0.1}) # default: 0.05
```

Grace time is the amount of time after a sync-action (pause or jump) that the slave will wait before performing any new syncing actions (specified in seconds).

```python
receiver = Receiver(player, {'grace_time': 1.0}) # default: 3.0
```

Jump ahead time is the amount of time that the slave will jump AHEAD of the master's playback position when slacking behind (or too far ahead). This gives the slave some time to process the (key-)frames for the new position before having to resume playback. (Specifed in seconds.)

```python
receiver = Receiver(player, {'jump_ahead': 2.5}) # default: 3.0
```





