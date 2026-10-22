robomas_plugins
====
## Description
It is for my Debug_CAN board.
It works on ros2 Humble Hawksbill and Foxy Fitzroy. (Using Foxy is not recommended.)

This supports only environment as 1 byte of 8 bits. (uint8_t == unsigned char)

## Usage

TODO: add example.

## Install

copy udev rule
```
sudo cp ~/ros2_ws/src/robomas_plugins/udev/60-robomas.rules /etc/udev/rules.d/60-robomas.rules
sudo udevadm control --reload-rules && sudo udevadm trigger
```

## Zero position

`robomas_zero_position` (`robomas_plugins/srv/ZeroPosition`) makes the current position of a motor in position control (mode 2) its new zero.
`motor` is 1..8, the same number as the `robomas_target<motor>` topic.

```
ros2 service call /robomas_zero_position robomas_plugins/srv/ZeroPosition "{motor: 1}"
```

The board has no command to set the origin; it takes the position at which position control was enabled as zero.
So the bridge resends the last `RobomasFrame` of that motor as disable (mode 0) and then as it was (mode 2), and sends target 0.
It fails if the last `RobomasFrame` sent to the motor was not mode 2.

The caller must switch its own targets to the new origin. A target computed for the old origin, sent after this, moves the motor by that much.
