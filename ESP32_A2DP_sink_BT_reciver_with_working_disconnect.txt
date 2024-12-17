#include "BluetoothA2DPSink.h"  // Library for Bluetooth A2DP sink functionality
#include "driver/i2s.h"         // Library for I2S communication (inter-integrated sound)

// Create an instance of the A2DP sink to receive Bluetooth audio streams
BluetoothA2DPSink a2dp_sink;

// I2S configuration parameters
#define I2S_NUM         I2S_NUM_0          // Define I2S number (I2S_NUM_0 or I2S_NUM_1)
#define I2S_SAMPLE_RATE 44100              // Set audio sample rate (44.1 kHz standard for audio)
#define I2S_BITS        I2S_BITS_PER_SAMPLE_16BIT // Set audio bit depth to 16-bit
#define I2S_CHANNELS    I2S_CHANNEL_FMT_RIGHT_LEFT // Set stereo audio (2 channels: left and right)
#define I2S_PIN_DATA    33                 // Define pin for I2S data (DOUT)
#define I2S_PIN_BCLK    25                 // Define pin for I2S bit clock (BCK)
#define I2S_PIN_LRCLK   26                 // Define pin for I2S left-right clock (LRCK)
#define NMUTE_PIN       32                 // Pin for controlling mute (to ensure audio output is not muted)

// Setup I2S communication: Configures I2S interface for audio streaming
void setup_i2s() {
    // Define the I2S configuration structure
    i2s_config_t i2s_config = {
        .mode = (i2s_mode_t)(I2S_MODE_MASTER | I2S_MODE_TX),  // Set I2S mode to Master and Transmitter
        .sample_rate = I2S_SAMPLE_RATE,  // Set the audio sample rate (44100 Hz)
        .bits_per_sample = I2S_BITS,    // Set the bit depth (16-bit per sample)
        .channel_format = I2S_CHANNELS, // Set stereo audio output (right-left channels)
        .communication_format = I2S_COMM_FORMAT_I2S_MSB,  // Use I2S MSB (Most Significant Bit) format
        .intr_alloc_flags = 0,          // No interrupt allocation flags
        .dma_buf_count = 8,             // Number of DMA buffers (used for efficient data transfer)
        .dma_buf_len = 64,              // Length of each DMA buffer (in bytes)
        .use_apll = false               // Disable APLL (Audio PLL) for sample rate generation
    };

    // Define I2S pin configuration
    i2s_pin_config_t pin_config = {
        .bck_io_num = I2S_PIN_BCLK,  // Pin for bit clock (BCK)
        .ws_io_num = I2S_PIN_LRCLK,  // Pin for left-right clock (LRCK)
        .data_out_num = I2S_PIN_DATA,  // Pin for data output (DOUT)
        .data_in_num = I2S_PIN_NO_CHANGE  // No input data (as it's a transmit-only setup)
    };

    // Install and configure the I2S driver with the provided settings
    i2s_driver_install(I2S_NUM, &i2s_config, 0, NULL);
    
    // Set the I2S pins based on the configuration
    i2s_set_pin(I2S_NUM, &pin_config);
}

// Callback function to process and send received audio data to I2S for output
void read_data_stream(const uint8_t *data, uint32_t length) {
    size_t bytes_written;  // Variable to store the number of bytes successfully written to I2S

    // Write the audio data to I2S for transmission to the external audio hardware
    i2s_write(I2S_NUM, data, length, &bytes_written, portMAX_DELAY);
}

// Callback function to handle Bluetooth disconnection or stream stop
void on_bluetooth_disconnect() {
    // Clear the I2S buffer by writing zeros
    i2s_zero_dma_buffer(I2S_NUM);

    // Mute the audio output
    digitalWrite(NMUTE_PIN, LOW);  // Set mute pin LOW to mute
}

// Callback function to handle Bluetooth reconnection or when data starts streaming
void on_bluetooth_reconnect() {
    // Unmute the audio output
    digitalWrite(NMUTE_PIN, HIGH);  // Set mute pin HIGH to unmute
}

// Setup function: Runs once when the ESP32 starts
void setup() {
    // Configure the NMUTE_PIN to ensure the audio output is not muted
    pinMode(NMUTE_PIN, OUTPUT);        // Set the mute pin as an output
    digitalWrite(NMUTE_PIN, HIGH);     // Set it HIGH to unmute (audio output enabled)

    // Initialize I2S interface with the configured settings
    setup_i2s();

    // Set up the A2DP sink to use the custom stream reader (callback function)
    a2dp_sink.set_stream_reader(read_data_stream, false);

    // Register the callback for Bluetooth disconnection
    a2dp_sink.set_on_data_received(on_bluetooth_reconnect);  // Triggered when data starts streaming

    // Start the A2DP sink with the device name "GOLF G60" and no authentication
    a2dp_sink.start("GOLF G60", false);
}

// Main loop: Runs continuously after setup
void loop() {
    // Check if the Bluetooth is disconnected manually (for example, by using a status check)
    if (!a2dp_sink.is_connected()) {
        on_bluetooth_disconnect();  // Call the disconnect handler
    }

    delay(100);  // Wait for 1 second (no specific action in the loop)
}
