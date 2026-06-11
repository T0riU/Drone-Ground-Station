# Drone Ground Station

This is a university diploma project. The code works but was never written to be reused or built upon. Threading is particularly rough, there is no proper synchronization, no graceful shutdown, and background threads can hang or silently crash. If you are looking for something to learn from or build on top of, this is probably not the right place to start.

A desktop ground station for controlling and monitoring a drone over MAVLink. You connect to the drone over Wi-Fi, and three windows open showing a live HUD, all sensor readings, and a basic control panel.

## What it does

You start the app, type in the drone's IP and port, and hit Connect. The app tries to reach the drone over MAVLink and waits for a heartbeat. If it gets one, the main interface opens. If something goes wrong you get a popup telling you what happened.

Once connected, telemetry starts streaming every 100ms into three separate windows sitting side by side on your screen. You can see the artificial horizon and compass live, check all the raw sensor numbers, and send basic flight commands like takeoff, land, and go to a GPS position.

## The three windows

### HUD

The main flight display sitting in the center of your screen. It shows an artificial horizon that tilts with the drone's roll and pitch, a scrolling compass bar at the top, and altitude and speed rulers on the sides. Everything updates live from the drone's attitude and navigation data.

### Sensors

A panel on the left showing all the raw numbers coming from the drone. GPS coordinates and altitude, battery voltage, current, and remaining charge, roll, pitch, and yaw from the IMU, heading and ground speed from the compass, temperature, and navigation output values.

### Control

A small panel on the right where you can actually fly the drone. You type in an altitude and hit Takeoff, hit Land to bring it down, or enter latitude, longitude, and altitude and hit Set Position to send the drone somewhere. There is a status bar at the top that shows what the drone is currently doing or any errors that come back.

## How the connection and commands work

The app connects using pymavlink and receives MAVLink messages in a background thread that runs continuously. Commands from the control panel go into a queue and get picked up by a separate command thread, so the UI never freezes waiting for the drone to respond.

When you hit Takeoff the app arms the drone first, waits for confirmation, then sends the takeoff command and watches altitude until the drone reaches the target. Landing works the same way in reverse and automatically disarms after touchdown. Mode changes retry up to three times before giving up.

## Project structure

```
main.py                  everything, split into classes
hud.html                 artificial horizon, compass, altitude and speed tapes
data.html                live sensor readout panel
control.html             takeoff, land, and set position controls

old/                     earlier versions kept for reference
```

### Classes in main.py

TelemetryApp handles the initial connection form where you enter IP and port before anything else opens.

TelemetryManager polls the drone every cycle and fills a dictionary with the latest GPS, battery, IMU, compass, temperature, and nav data.

GUIManager creates the three webview windows and positions them on screen relative to the display size.

CommandHandler sits in a loop waiting for commands from the queue and calls the right DroneController method for each one.

DroneController wraps all the MAVLink flight operations: arm, disarm, takeoff, land, change mode, and set position. It talks to StatusManager so the control window always shows what is happening.

MainController wires everything together, starts the threads, and pushes telemetry updates into the webview windows every 100ms via JavaScript calls.

## Requirements

```
pymavlink
pywebview
jinja2
tkinter
```

The app currently uses Windows APIs to position windows on screen so it runs on Windows only as is.

## Setup

Install the dependencies, make sure all four files are in the same folder, and run main.py. The default IP is 192.168.160.1 and the default port is 14550 which matches a typical ArduPilot Wi-Fi setup. Change them in the connection form if your drone uses different values.
