---
description: >-
  Basic setup that allows you to log topic (based on stroke sensor, you can
  adapt this to log other values).
---

# ROS2 Systems Setup with Foxglove Studio and mcap logging

First things first, you need data coming. Be able to log into `Serial.print()` monitor in Arduino IDE's serial monitor or any TTY enabled terminal.

Then you have to make sure microROS is installed within the Arduino library. You can make sure of it here, (you can use your host computer to do this, does not necessarily have to be the one running ROS2). You can get the installation guide here: [https://www.hackster.io/514301/micro-ros-on-esp32-using-arduino-ide-1360ca](https://www.hackster.io/514301/micro-ros-on-esp32-using-arduino-ide-1360ca)&#x20;

Reminder: use documentation from IPST (which I've shared previosly on Discord) for wiring.

Install the compatible version of ROS2 in Raspberry Pi. (Note: get monitor, keyboard and a mouse for raspberry pi while doing this).

Confirm if Raspberry Pi has Ubuntu 24.04 version. Then, install ROS2 through this documentation: [https://docs.ros.org/en/jazzy/index.html](https://docs.ros.org/en/jazzy/index.html)&#x20;

### Publishing topics

Modify broilerplate code from hackster.io to suit your needs as in this example.

```cpp
#include <micro_ros_arduino.h>
#include <stdio.h>
#include <rcl/rcl.h>
#include <rcl/error_handling.h>
#include <rclc/rclc.h>
#include <rclc/executor.h>
#include <std_msgs/msg/int32.h>

rcl_publisher_t publisher;
std_msgs__msg__Int32 msg;
rclc_executor_t executor;
rclc_support_t support;
rcl_allocator_t allocator;
rcl_node_t node;
rcl_timer_t timer;

#define LED_PIN 2
#define ADC_PIN 34  // ADC pin connected to the potentiometer

#define RCCHECK(fn) { rcl_ret_t temp_rc = fn; if((temp_rc != RCL_RET_OK)){error_loop();}}
#define RCSOFTCHECK(fn) { rcl_ret_t temp_rc = fn; if((temp_rc != RCL_RET_OK)){}}

void error_loop(){
  while(1){
    digitalWrite(LED_PIN, !digitalRead(LED_PIN));
    delay(100);
  }
}

void timer_callback(rcl_timer_t * timer, int64_t last_call_time)
{  
  RCLC_UNUSED(last_call_time);
  if (timer != NULL) {
    int adc_value = analogRead(ADC_PIN);  // Read analog value
    msg.data = adc_value;
    RCSOFTCHECK(rcl_publish(&publisher, &msg, NULL));
  }
}

void setup() {
  set_microros_transports();

  pinMode(LED_PIN, OUTPUT);
  digitalWrite(LED_PIN, HIGH);  

  pinMode(ADC_PIN, INPUT);  // Initialize ADC pin

  delay(2000);

  allocator = rcl_get_default_allocator();

  // Create init_options
  RCCHECK(rclc_support_init(&support, 0, NULL, &allocator));

  // Create node
  RCCHECK(rclc_node_init_default(&node, "micro_ros_arduino_node", "", &support));

  // Create publisher
  RCCHECK(rclc_publisher_init_default(
    &publisher,
    &node,
    ROSIDL_GET_MSG_TYPE_SUPPORT(std_msgs, msg, Int32),
    "adc_reading"));

  // Create timer
  const unsigned int timer_timeout = 1000;  // 1 second
  RCCHECK(rclc_timer_init_default(
    &timer,
    &support,
    RCL_MS_TO_NS(timer_timeout),
    timer_callback));

  // Create executor
  RCCHECK(rclc_executor_init(&executor, &support.context, 1, &allocator));
  RCCHECK(rclc_executor_add_timer(&executor, &timer));
}

void loop() {
  delay(100);
  RCSOFTCHECK(rclc_executor_spin_some(&executor, RCL_MS_TO_NS(100)));
}
```

This code is for logging potentiometer used to measure suspension displacement.

Modify this code to record RPM and electrical signal (just multiple topics) for Dyno testing.

### Foxglove Studio

It is similar to Rviz, but with much more interactivity available. It can be used as dashboard for the rest of 2026 season. [https://www.foxglove.dev/](https://app.foxglove.dev/)&#x20;

You need to set this up on raspberry pi and listen to topics using the studio.

### mcap file

Following this tutorial, you will be able to record events (topics) in mcap file: (Raspberry Pi) [https://mcap.dev/guides/getting-started/ros-2](https://mcap.dev/guides/getting-started/ros-2)

## Expected deliverables

1. ROS topics list via `ros2 topic list` command on Raspberry Pi terminal. Take screenshot.
2. Screenshot from Foxglove Studio recording those topics. (PNG)
3. mcap file from demo run.

You can use the demo RPM tester kit to simulate RPM situation (I made that last semester, look around ME Control laboratory for the references).&#x20;

Upload those files on BlackPearl's Google Drive (Create a folder called SYSTEMS and sub-folder within in with DATA\_\<date> in YYYYMMDD format.

