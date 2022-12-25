# motion-activated_brightness_3times
Homeassistant Motion Activated with 3 Times 
 
 blueprint:
  name: Motion-activated Light-brightness
  description: Turn the light on to brightness when motion is detected, depending on the selected time period, different brightness and turn-on time.
  domain: automation
  input:
    motion_entity:
      name: Motion Sensor
      selector:
        entity:
          domain: binary_sensor
          device_class: motion
    light_target:
      name: Light
      selector:
        target:
          entity:
            domain: light
    time1 #on_night_time:
      name: (Required) On Time Night
      description: The time when the night mode starts.
      selector:
        time: {}
    time2 #off_night_time:
      name: (Required) Off Time Night
      description: The time when the night mode ends.
      selector:
        time: {}
    time3 off_night_time:
      name: (Required) Off Time Night
      description: The time when the night mode ends.
      selector:
        time: {}
    time1_brightness day_brightness:
      name: Day brightness
      description: Brightness, in daytime mode
      default: 1
      selector:
        number:
          min: 1.0
          max: 100.0
          step: 1.0
          mode: slider
    time2_brightness night_brightness:
      name: Night brightness
      description: Brightness, in night mode mode
      default: 1
      selector:
        number:
          min: 1.0
          max: 100.0
          step: 1.0
          mode: slider
    time3_brightness night_brightness:
      name: Night brightness
      description: Brightness, in night mode mode
      default: 1
      selector:
        number:
          min: 1.0
          max: 100.0
          step: 1.0
          mode: slider
    no_motion_wait_time1 #no_motion_wait_day:
      name: Wait time day
      description: Time to leave the light on after detecting the last movement in mode 1.
      default: 120
      selector:
        number:
          min: 0
          max: 3600
          unit_of_measurement: seconds
    no_motion_wait_time2 #no_motion_wait_night:
      name: Wait time night
      description: Time to leave the light on after detecting the last movement in mode 2.
      default: 120
      selector:
        number:
          min: 0
          max: 3600
          unit_of_measurement: seconds
    no_motion_wait_time3 #no_motion_wait_night:
      name: Wait time night
      description: Time to leave the light on after detecting the last movement in mode 3.
      default: 120
      selector:
        number:
          min: 0
          max: 3600
          unit_of_measurement: seconds

# If motion is detected within the delay,
# we restart the script.
mode: restart
max_exceeded: silent

trigger:
  platform: state
  entity_id: !input motion_entity
  from: "off"
  to: "on"

action:
  - choose:
      - conditions:
          - condition: time
            after: !input on_night_time
            before: !input off_night_time
        sequence:
          - alias: "Turn on the light"
            service: light.turn_on
            data:
              brightness_pct: !input night_brightness
            target: !input light_target
          - alias: "Wait until there is no motion from device"
            wait_for_trigger:
              platform: state
              entity_id: !input motion_entity
              from: "on"
              to: "off"
          - alias: "Wait the number of seconds that has been set"
            delay: !input no_motion_wait_night
          - alias: "Turn off the light"
            service: light.turn_off
            target: !input light_target
    default:
      - alias: "Turn on the light"
        service: light.turn_on
        data:
          brightness_pct: !input day_brightness
        target: !input light_target
      - alias: "Wait until there is no motion from device"
        wait_for_trigger:
          platform: state
          entity_id: !input motion_entity
          from: "on"
          to: "off"
      - alias: "Wait the number of seconds that has been set"
        delay: !input no_motion_wait_day
      - alias: "Turn off the light"
        service: light.turn_off
        target: !input light_target
