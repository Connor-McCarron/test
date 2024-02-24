 ----------------------------------------------------------------
# Import the Adafruit Bluetooth library, part of Blinka.  Technical reference:
# https://circuitpython.readthedocs.io/projects/ble/en/latest/api.html

from adafruit_ble import BLERadio
from adafruit_ble.advertising.standard import ProvideServicesAdvertisement
from adafruit_ble.services.nordic import UARTService

# ----------------------------------------------------------------
# Initialize global variables for the main loop.
ble = BLERadio()
uart_connection = None

# ----------------------------------------------------------------
# Begin the main processing loop.

while True:
    if not uart_connection:
        print("Trying to connect...")
        # Check for any device advertising services
        for adv in ble.start_scan(ProvideServicesAdvertisement):
            # Print name of the device
            name = adv.complete_name
            if name:
                print(name)
            # Print what services that are being advertised
            for svc in adv.services:
                print(str(svc))
            # Look for UART service and establish connection
            if UARTService in adv.services:
                uart_connection = ble.connect(adv)
                print("Connected")
                break
        ble.stop_scan()
    # Once connected start receiving data
    if uart_connection and uart_connection.connected:
        uart_service = uart_connection[UARTService]
        while uart_connection.connected:
            print(uart_service.readline().decode("utf-8").rstrip())
