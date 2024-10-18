<p align="center">
  <a href="" rel="noopener">
 <img width=200px height=200px src="https://github.com/deco3801-mortein/frontend/blob/main/src/assets/img/Logo.png" alt="Project logo"></a>
</p>

<h3 align="center">VibeGrow</h3>

---

## 🌺 Table of Contents

- [About](#about)
- [Our Features](#our_features)
- [Getting Started](#getting_started)
- [Built Using](#built_using)
- [Authors](#authors)

## 🍀 About <a name = "about"></a>

Access our web application here: https://vibegrow.pro/ 

Welcome to VibeGrow! Our innovative project combines hardware and software to revolutionize plant care. At its core is a small device attached to the base of a plant, capable of producing vibrations at frequencies used for insect communication. This device is wirelessly operated through a user-friendly web interface.
Beyond pest control, VibeGrow monitors crucial plant health metrics (soil moisture, temperature, and light exposure), allowing users to care for their plants remotely. Our web application also features comprehensive plant and pest search functionality, providing users with easy access to vital gardening information.

## 🌷 Our Features<a name = "our_features"></a>

- 💫 **Customized Homepage**: Displays all user plants with names and icons for quick identification.
- 💫 **Real-Time Plant Health Monitoring:** Three progress bars (temperature, light exposure, soil moisture) provide at-a-glance health status.
- 💫 **Detailed Garden Guidance:** Separate search functionalities for plants and pests, offering cultivation tips and pest control advice.
  - Pest information gathered from [The Royal Horticultural Society](https://www.rhs.org.uk/biodiversity) and [GrowVeg](https://www.growveg.com.au/pests/australia-and-nz/).
  - Plant information retrieved from the [Perenual Plant API](https://perenual.com/docs/api).

## 🙉 Getting Started <a name = "getting_started"></a>
All code for this project can be found [here](https://github.com/deco3801-mortein).


# Embedded System
The [embedded repository](https://github.com/deco3801-mortein/embedded) contains the firmware for the physical device.

## Build
This folder must be opened in Visual Studio Code with the PlatformIO IDE extension installed.
Wait for the extension to initialise, and open the PlatformIO Side Bar from the Activity Bar
on the far left-hand side. You may need to reload the window on the first attempt.

For security purposes, the `/include/secrets.h` file is not included in this repository.
This contains the necessary authentication the connection to the MQTT broker running on
AWS servers. Please create this file, and fill in the required keys and certificates.

The template for this `/include/secrets.h` file is demonstrated below:
```c
#ifndef SECRETS_H
#define SECRETS_H

#include <pgmspace.h>

// URL for AWS MQTT broker
static const char AWS_IOT_ENDPOINT[] = "www.fill-in-mqtt-broker-url-here.com";
static const unsigned short AWS_IOT_MQTT_PORT = 8883;
 
// Amazon Root CA 1
static const char AWS_ROOT_CA1[] PROGMEM = R"EOF(
-----BEGIN CERTIFICATE-----
Copy AWS_ROOT_CA here.
-----END CERTIFICATE-----
)EOF";

// Device Certificate
static const char AWS_CERT[] PROGMEM = R"KEY(
-----BEGIN CERTIFICATE-----
Copy device certificate here.
-----END CERTIFICATE-----
)KEY";
 
// Device Private Key
static const char AWS_PRIVATE_KEY[] PROGMEM = R"KEY(
-----BEGIN RSA PRIVATE KEY-----
Copy device private key here.
-----END RSA PRIVATE KEY-----
)KEY";

#endif
```

The firmware should now be ready. In the PlatformIO Side Bar, select
`final_prototype > General > Build` to build.

## Upload

Connect the Arduino Nano ESP32 to your computer via USB. Then, select
`final_prototype > General > Upload` from the PlatformIO Side Bar to flash the firmware to the
device.

To see basic logging information of the device's operation, select
`final_prototype > General > Monitor` from the PlatformIO Side Bar while the device is still
connected over USB. This will print all Serial logging messages to the console.

</br>

# API

In order to authenticate to the AWS MQTT broker, pass a client certificate and private key into
the following command to generate the required PFX file:

The environment required to build and run this application is defined by this repository's
dev container [configuration file](https://github.com/deco3801-mortein/api/blob/main/.devcontainer/devcontainer.json).

Either open this repository in a dev container using a
[supporting tool](https://containers.dev/supporting) or replicate the required environment manually.

## Deployment

Create an AWS account and authenticate to the account from your working environment.

Create an IAM role named `lambda-role` with the following AWS managed permissions policies attached:

-   AmazonS3ReadOnlyAccess
-   AWSLambdaBasicExecutionRole

Create an RDS database instance using version 16.4 of the PostgreSQL engine.

Create an AWS IoT certificate with the following policies attached:

-   iot:Connect
-   iot:Publish
-   iot:Receive
-   iot:Subscribe

Generate a PFX certificate from the AWS IoT private key and certificate using the following command:

```
openssl pkcs12 -export -out api.pfx -inkey private.pem.key -in certificate.pem.crt
```

Create an S3 bucket named `api-mqtt-certificate` and upload the generated `api.pfx` file.

From within the `Mortein` directory, deploy the API using the `dotnet lambda deploy-serverless`
command, passing the `--template-parameters` argument with the parameters defined in the
_Parameters_ section of the `serverless-template.json` file. These parameters come from the hostname
and credentials of your RDS instance, as well as the domain name of the default AWS IoT domain
configuration. The MQTT client ID can be any string.
</br>

# Queue Handler 

## Overview
A lambda function that takes payload data from an IoT Core Rule invocation and adds it to an RDS database. It uses a custom serializer to take in the payload data and converts it to a class from the Mortein.Types package. Then EF Core is used to upload the data to the database.

## Setup
1. Clone the repo and start the dev container.
2. Ensure you have an active github token to authenticate to Github Packages, a guide can be found here: https://docs.github.com/en/packages/learn-github-packages/about-permissions-for-github-packages.
3. Deploy the lambda function to AWS using `dotnet lambda deploy-function`
4. Create an IoT Core rule that is triggered with your chosen topic (# for wildcard) which invokes the deployed function. A sample is provided below:
    ```
    SELECT 
    DeviceId,
    Timestamp,
    Moisture,
    Sunlight,
    IsVibrating,
    Temperature
    FROM 'iot/test'
    ```
# Web Application

-   Clone or download the [frontend repository](https://github.com/deco3801-mortein/frontend) to your local machine.
-   Open the project folder using your preferred code editor
-   Add the following environment variables to your environment (these will give you access to our sdk package and the Perenual API for the plant guide):

    `NPM_AUTH=<your personal access token>`

    `VITE_API_KEY=<your perenual plant API key>`

    -   To get the NPM_AUTH token:
        -   Contact us to for access to install our package
        -   Authenticate with npm registry using a personal access token (link for help: https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-npm-registry)
    -   To get the VITE_API_KEY:
        -   Create an account and ask for a key from [perenual.com](https://perenual.com/docs/api).

-   In your terminal, navigate to the cloned or downloaded frontend repository folder and run the following commands:
    -   `npm install`
    -   `npm run build`
-   The created dist file can then be deployed using any web hosting service (such as AWS Amplify)
-   You will need to give the hosting service your Perenual API key

-   **Accessing Our Deployed Web Application**:
    -   Go to https://vibegrow.pro to see our deployed website using AWS Amplify


### Images Used

-   GuidancePage.jsx
    -   Plant in flower pot icon. © [alekseyvanin](https://stock.adobe.com/au/contributor/206204820/alekseyvanin?load_type=author&prev_url=detail) / Adobe Stock
    -   Vector set of icons with insects for pest control business. © [Alexandra Gl](https://stock.adobe.com/au/contributor/201966471/alexandra-gl?load_type=author&prev_url=detail) / Adobe Stock
-   HomePage.jsx
    -   Set of trendy potted plants for home. © [Good Studio](https://stock.adobe.com/au/contributor/206710010/good-studio?load_type=author&prev_url=detail) / Adobe Stock


## 🐜 Built Using <a name = "built_using"></a>

- **Frontend** - React, Vite, AWS Amplify, Perenual Plant API
- **Backend** - AWS IoT Core MQTT Broker, an RDS Database
- **Embedded Development** - ESP32 microcontroller (Arduino Uno ESP32), Capacitive Soil Moisture Sensor,​Linear resonant actuator with motor driver,​Light intensity sensor ,
- ​Temperature and humidity sensor,​LED circuit board indicator bar, ​Green LED and RGB LED, ​MAX7219 LED matrix driver IC, Pushbutton, Switch, ​4×AAA battery holder 

## ✍️ Authors <a name = "authors"></a>

- ✨**Amelia Caddie** - Frontend Development
- ✨**Denzel Strauss** - Backend Development
- ✨**Dominic Pincus** - Embedded Development
- ✨**Michael Jones** - Embedded Development
- ✨**Mingkun Li** - UI design & Frontend Development
- ✨**William Sawyer** - Backend Development
