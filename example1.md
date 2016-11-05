# Serial tutorial - Arduino multi-data

Here's an example of sending multiple pieces of data from the Arduino to the Photon. It's fake weather data, consisting of 7 numbers. 

It uses snprintf on the Arduino side to convert the data to text, and sscanf on the Photon side to read the data back.

## Arduino code

```
// Arduino code

// Constants

// Structures
typedef struct {
  int temperature;
  int humidity;
  int pressure;
  int luminosity;
  int winddir;
  int windspeed;
  int rainfall;
} WeatherData;

// Forward declarations
void getWeatherData(WeatherData &data);
void sendWeatherData(const WeatherData &data);

// Global variables
char sendBuf[256];

void setup() {
  // Serial TX (1) is connected to Photon RX
  // Serial RX (0) is connected to Photon TX
  // Ardiuno GND is connected to Photon GND
  Serial.begin(19200);
}

void loop() {
  WeatherData data;
  getWeatherData(data);
  sendWeatherData(data);
  delay(5000);
}

void getWeatherData(WeatherData &data) {
  // This just generates random data for testing
  data.temperature = rand();
  data.humidity = rand();
  data.pressure = rand();
  data.luminosity = rand();
  data.winddir = rand();
  data.windspeed = rand();
  data.rainfall = rand();
}

void sendWeatherData(const WeatherData &data) {

  snprintf(sendBuf, sizeof(sendBuf), "%d,%d,%d,%d,%d,%d,%d\n",
      data.temperature, data.humidity, data.pressure, data.luminosity, data.winddir, data.windspeed, data.rainfall);
  Serial.print(sendBuf);
}


```


## Photon code

```
#include "Particle.h"

// Constants
const size_t READ_BUF_SIZE = 256;

// Structures
typedef struct {
	int temperature;
	int humidity;
	int pressure;
	int luminosity;
	int winddir;
	int windspeed;
	int rainfall;
} WeatherData;

// Forward declarations
void processBuffer();
void handleWeatherData(const WeatherData &data);

// Global variables
int counter = 0;
unsigned long lastSend = 0;

char readBuf[READ_BUF_SIZE];
size_t readBufOffset = 0;

void setup() {
	Serial.begin(9600);

	// Serial1 RX is connected to Arduino TX (1)
	// Serial2 TX is connected to Arduino RX (0)
	// Photon GND is connected to Arduino GND
	Serial1.begin(19200);
}

void loop() {

	// Read data from serial
	while(Serial1.available()) {
		if (readBufOffset < READ_BUF_SIZE) {
			char c = Serial1.read();
			if (c != '\n') {
				// Add character to buffer
				readBuf[readBufOffset++] = c;
			}
			else {
				// End of line character found, process line
				readBuf[readBufOffset] = 0;
				processBuffer();
				readBufOffset = 0;
			}
		}
		else {
			Serial.println("readBuf overflow, emptying buffer");
			readBufOffset = 0;
		}
	}

}

void processBuffer() {
	// Serial.printlnf("Received from Arduino: %s", readBuf);
	WeatherData data;

	if (sscanf(readBuf, "%d,%d,%d,%d,%d,%d,%d", &data.temperature, &data.humidity, &data.pressure,
			&data.luminosity, &data.winddir, &data.windspeed, &data.rainfall) == 7) {

		handleWeatherData(data);
	}
	else {
		Serial.printlnf("invalid data %s", readBuf);
	}
}

void handleWeatherData(const WeatherData &data) {
	Serial.printlnf("got temperature=%d humidity=%d pressure=%d luminosity=%d winddir=%d windspeed=%d rainfall=%d",
			data.temperature, data.humidity, data.pressure, data.luminosity, data.winddir, data.windspeed, data.rainfall);
}



```