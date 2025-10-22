# Smart Pill Dispenser System

A comprehensive web-based management interface for an intelligent robotic pill dispensing system. This project integrates hardware control, database management, and user interfaces to create an automated pharmacy solution that dispenses medications accurately and securely.

## Overview

The Smart Pill Dispenser System provides a complete solution for automated medication management. The system features a Flask-based web application that interfaces with Raspberry Pi hardware to control a robotic dispensing mechanism. Users can browse medications by ailment category, place orders, and receive precisely measured doses through an automated dispensing process.

## Screenshots

![System Interface](docs/images/ui.jpeg)

Add your screenshots to the `docs/images/` folder and reference them here. Common screenshots to include:

* Home page interface
* Ailment selection screen
* Medication browsing interface
* Admin portal
* Physical hardware setup

## System Architecture

### Web Application

The core of the system is a Flask web server that provides:

* User-facing medication browsing and ordering interface
* Administrative portal for inventory management
* Real-time communication with hardware via MQTT
* SQLite database for persistent storage
* Password-protected access control
* QR code-based medication identification

### Hardware Integration

The system controls multiple hardware components through Raspberry Pi GPIO:

* **Stepper Motor Control**: Utilizes an Ender 3D printer's stepper motors for precise positioning along the X-axis to navigate between medication containers
* **DC Motor**: Controls a blade mechanism for cutting pill blister packs
* **Solenoid Lock**: Provides physical security by locking/unlocking the dispenser cabinet
* **IR Beam Sensors**: Counts individual pills as they pass through the dispensing mechanism
* **LED Indicators**: Provides visual feedback during the refill process
* **Serial Communication**: Interfaces with hardware components via USB serial connections

### Database Schema

The system uses SQLAlchemy ORM with three main models:

* **User**: Stores administrator credentials with hashed passwords
* **Pharmacy**: Represents physical dispenser locations
* **Medication**: Contains medication details including name, quantity, dosage, description, image, and functional category

## Features

### Customer Interface

**Ailment-Based Selection**: Users browse medications by their therapeutic purpose (pain relief, allergy relief, fever reduction, etc.) rather than by specific drug names, making the system more intuitive for non-medical users.

**Visual Product Display**: Each medication is presented with images, descriptions, and dosage information to help users make informed decisions.

**Order Processing**: The system handles order submission, inventory updates, and automatic dispensing through MQTT messaging.

**Order Confirmation**: Users receive immediate feedback when their order is successfully placed and being processed.

### Administrative Features

**Secure Access**: Password-protected admin portal with bcrypt hashed credentials ensures only authorized personnel can manage inventory.

**Medication Management**: Administrators can replace medications in the dispensing system with guided location identification.

**QR Code Scanning**: Automatic medication identification through QR codes streamlines the refill process and reduces manual data entry errors.

**LED-Guided Refilling**: When replacing medication, the system illuminates the specific storage location with LEDs to prevent errors.

**Inventory Tracking**: Real-time monitoring of medication quantities with automatic updates after each dispensing operation.

**Solenoid Lock Control**: Automatic locking and unlocking of the dispensing cabinet to maintain security when not in admin mode.

## Technical Implementation

### Dispensing Algorithm

The `advance.py` module implements a sophisticated multi-step dispensing process:

1. **Home Position**: The stepper motor carriage moves to the home position (0,0,0) for initialization
2. **Navigation**: The system calculates and moves to the specific X-position corresponding to the requested medication slot
3. **Blade Control**: A DC motor-controlled blade opens the blister pack access
4. **Dispensing**: A relay-controlled motor activates the dispensing mechanism
5. **Counting**: IR beam sensors count pills as they pass through, ensuring accurate quantity
6. **Cutting**: The blade mechanism closes to separate the dispensed pills
7. **Return**: The system returns to home position and cleans up GPIO resources

### MQTT Communication

The system uses MQTT publish/subscribe messaging for asynchronous communication between the web interface and hardware control:

* **Order Topic**: Publishes dispensing requests with location and quantity
* **Done Topic**: Receives completion notifications from the hardware controller
* **Broker**: Runs on localhost:1883 for low-latency communication

### Serial Communication

Two serial connections manage different hardware systems:

* **Primary Serial (USB1)**: Communicates with the Ender 3D printer controller at 115200 baud for G-code commands
* **QR Serial (USB0)**: Interfaces with the QR code scanner at 38400 baud for medication identification

### Web Scraping for QR Codes

The `scan.py` module implements a robust QR code reading system that:

1. Reads QR code data from the serial scanner
2. Makes HTTP requests to the embedded URL
3. Handles advertisement redirects automatically
4. Extracts medication data using BeautifulSoup
5. Returns structured JSON data for database insertion

## Installation

### Hardware Requirements

* Raspberry Pi (tested on Raspberry Pi 4)
* Ender 3D printer controller board for stepper motor control
* DC motor driver (MDD10A compatible)
* IR break beam sensors
* 5V relay module
* Solenoid lock mechanism
* QR code scanner module
* LED indicators

### Software Requirements

Install the required Python packages:

```bash
pip install -r requirements.txt
```

Required packages include:
* Flask and Flask extensions (SQLAlchemy, Login, WTF, Migrate)
* Paho MQTT client
* PySerial for serial communication
* BeautifulSoup4 for web scraping
* Pyzbar for QR code decoding
* RPi.GPIO for Raspberry Pi GPIO control
* NumPy for numerical operations

### Database Setup

Initialize the database by uncommenting the database creation code in `flaskServer.py`:

```python
with app.app_context():
    db.create_all()
```

Run the server once to create the database, then comment out the code again to prevent reinitialization.

### Hardware Configuration

Connect the hardware components to the following GPIO pins (BCM mode):

* Solenoid Lock: GPIO 21
* DC Motor Control: GPIO 23 (direction), GPIO 24 (enable)
* IR Sensors: GPIO 27, 22, 5, 6 (for positions 1-4)
* Dispenser Motors: GPIO 4, 17, 13, 19 (for positions 1-4)

Serial devices should be connected at:
* `/dev/ttyUSB1` for the printer controller
* `/dev/ttyUSB0` for the QR scanner

## Usage

### Starting the Server

Run the Flask application:

```bash
python flaskServer.py
```

The server will start on `http://0.0.0.0:5050` and be accessible from any device on the network.

### Customer Workflow

1. Navigate to the home page and click "Shop Now"
2. Select a medication category based on your ailment
3. Review the available medications in that category
4. Click "Submit Order" to dispense the medication
5. Wait for the order success confirmation

### Administrative Workflow

1. Click "Admin" from the home page
2. Enter the administrator password
3. Select the medication slot to refill
4. Scan the QR code on the new medication package
5. The system reads medication information and updates the database
6. Physically refill the indicated slot (LED will illuminate)
7. Confirm completion to return to the admin portal

### iOS Application

An iOS companion app is included in the `PillDispenser` directory, built with SwiftUI. The app provides:

* Medication selection via picker wheel
* Quantity adjustment with stepper control
* Name entry for order tracking
* API integration with the Flask server

To build the iOS app, open `PillDispenser.xcodeproj` in Xcode.

## API Endpoints

### Public Endpoints

* `GET /` - Home page
* `GET /ailment` - Medication category selection
* `GET /place_order/<ailment>` - Browse medications by category
* `POST /place_order/<ailment>` - Submit an order
* `GET /order_success` - Order confirmation page

### Administrative Endpoints

* `POST /admin` - Admin login
* `GET /admin_portal` - Medication replacement interface
* `POST /admin_portal` - Select medication slot for replacement
* `GET /qr/<location>` - QR code scanning interface
* `POST /qr/<location>` - Process scanned medication data
* `GET /refill/<location>/<name>` - Physical refill confirmation
* `GET /lock` - Lock the solenoid
* `GET /unlock` - Unlock the solenoid

## Security Features

* Password-protected administrative access using Werkzeug security
* Bcrypt password hashing for credential storage
* Solenoid lock automatically engages when not in admin mode
* Flask CSRF protection via Flask-WTF
* Secret key configuration for session management

## Project Structure

```
PillDispensorProject/
├── flaskServer.py          # Main Flask application
├── advance.py              # Dispensing control logic
├── setup.py                # Basic dispensing without positioning
├── solenoid.py             # Lock/unlock control
├── scan.py                 # QR code reading and parsing
├── led.py                  # LED indicator control
├── requirements.txt        # Python dependencies
├── templates/              # HTML templates
│   ├── base.html          # Base template with styling
│   ├── home.html          # Landing page
│   ├── ailment.html       # Category selection
│   ├── indexProduc.html   # Product browsing
│   ├── order_success.html # Order confirmation
│   ├── admin.html         # Admin login
│   ├── index2.html        # Medication replacement
│   ├── QR.html            # QR scanning interface
│   └── replace.html       # Refill confirmation
├── static/                 # Product images
└── PillDispenser/          # iOS application
    └── PillDispenser/
        ├── ContentView.swift
        └── PillDispenserApp.swift
```

## Hardware Control Modules

Individual hardware control modules are provided for testing and development:

* `dcmotor.py` - DC motor control for blade mechanism
* `stepper.py` - Stepper motor testing
* `test_beam.py` - IR sensor testing
* `qr.py` - QR scanner testing
* `dispense.py` - Basic dispensing logic
* `slider.py` - Position control
* `cutting.py` - Blade control
* `ender.py` - 3D printer controller communication
* `fun.py` - Utility functions
* `helper.py` - Database helper functions

## Configuration

### Flask Configuration

The server is configured with:

* Secret key: `mysecret` (change in production)
* Database: SQLite at `instance/myDB.db`
* Server name: `127.0.0.1:5050`
* Application root: `/`
* URL scheme: `http`

### MQTT Configuration

* Broker address: `localhost`
* Port: `1883`
* Topics: `order`, `done`

### Medication Positions

The system supports 8 medication slots with predefined X-axis positions:

* Position 1: 5mm
* Position 2: 120mm
* Position 3: 240mm
* Position 4: 360mm
* Position 5: 500mm
* Position 6: 600mm
* Position 7: 700mm
* Position 8: 800mm

## Development Notes

### Logging

Printer communication can be logged by uncommenting the logging configuration in `flaskServer.py` and `advance.py`. Logs are written to `printer.log`.

### Testing Individual Components

Each hardware module can be run independently for testing:

```bash
python advance.py          # Test full dispensing cycle
python solenoid.py         # Test lock mechanism
python scan.py             # Test QR scanner
python test_beam.py        # Test IR sensors
```

### Database Access

The SQLite database can be accessed directly at `instance/myDB.db` for debugging or manual data manipulation.

## Future Enhancements

Potential improvements to the system include:

* Prescription verification and tracking
* Multi-user account system with individual medication profiles
* Scheduled medication reminders and automatic dispensing
* Integration with pharmacy inventory management systems
* Mobile notification system for refill alerts
* Expanded iOS app functionality with order history
* Camera integration for computer vision verification
* Integration with health monitoring devices
* Cloud-based data synchronization
* Analytics dashboard for usage patterns

## Troubleshooting

### Common Issues

**Serial Communication Errors**: Verify that the USB devices are connected to the correct ports (`/dev/ttyUSB0` and `/dev/ttyUSB1`). Check permissions with `sudo chmod 666 /dev/ttyUSB*`.

**GPIO Permission Errors**: Run the Flask server with appropriate permissions or add the user to the `gpio` group.

**MQTT Connection Failures**: Ensure the MQTT broker (Mosquitto) is installed and running: `sudo systemctl start mosquitto`.

**Database Errors**: If the database becomes corrupted, delete `instance/myDB.db` and reinitialize using the setup code.

**Stepper Motor Not Moving**: Verify G-code communication by checking the serial output. The printer controller must be properly initialized and connected.

**IR Sensor False Readings**: Adjust the sensor sensitivity or check for ambient light interference. Clean the sensor lenses if necessary.

## Contributors

Created by Ben Randoing

Initial release: February 12, 2023

## License

This project is provided as-is for educational and research purposes.

## Acknowledgments

This system demonstrates the integration of web technologies, embedded systems, and robotics to solve real-world healthcare automation challenges. The project showcases full-stack development including frontend UI design, backend server logic, database management, hardware interfacing, and mobile application development.
