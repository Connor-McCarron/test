import time
from adafruit_ble import BLERadio
from adafruit_ble.advertising.standard import ProvideServicesAdvertisement
from adafruit_ble.services.nordic import UARTService
import paho.mqtt.client as mqtt

# Initialize BLE and MQTT clients
ble = BLERadio()
uart_connection = None
mqtt_client = mqtt.Client()

# MQTT Broker Information
mqtt_broker = "localhost"  # Change this to the IP address or hostname of your MQTT broker
mqtt_port = 1883  # Default MQTT port

# Initialize MQTT client
mqtt_client = mqtt.Client()
mqtt_client.connect(mqtt_broker, mqtt_port)

# Function to handle MQTT publish
def publish_mqtt(topic, payload):
    mqtt_client.publish(topic, payload)

# Main loop
while True:
    if not uart_connection:
        print("Trying to connect...")
        # Check for any device advertising services
        for adv in ble.start_scan(ProvideServicesAdvertisement):
            # Look for UART service and establish connection
            if UARTService in adv.services:
                uart_connection = ble.connect(adv)
                print("Connected")
                break
        ble.stop_scan()

    # Once connected, start receiving data
    if uart_connection and uart_connection.connected:
        uart_service = uart_connection[UARTService]
        while uart_connection.connected:
            received_data = uart_service.read()
            if received_data is not None:
                # Forward received data over MQTT
                print("Received data from UART:", received_data.decode("utf-8"))
                publish_mqtt("sensor_data", received_data)
    else:
        print("Disconnected")
        uart_connection = None

    time.sleep(0.1)  # Short delay before next iteration
