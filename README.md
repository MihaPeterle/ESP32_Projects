# ESP32_A2DP_sink_to_I2S_PCM5102A_BT_reciver

This code is for the ESP32, programmed in the Arduino environment (version 1.8.18, but should also work with Arduino 2.X.X). It utilizes the I2S drivers provided by Arduino and includes the ESP32-A2DP library from pschatzmann (available at: https://github.com/pschatzmann/ESP32-A2DP).

The program is designed to leverage the ESP32's Bluetooth A2DP capabilities and the Arduino I2S drivers for audio data processing.

Problem: When device gets disconnected I2S plays last buffer of data it recived continiously.

Solution: ESP32_A2DP_sink_BT_reciver_with_working_disconnect (checking in while, sorry but no interupts here)
