# Reference Application Introduction:

This Reference Application is targetted specifically at the RD2541 Reference Design and is built on mbedOS and uses the "mbed-cloud-client" library to handle the Pelion Platform connection, registration and LWM2M resources. The application simulates a button press every 5 seconds and relays the packet to Arm Pelion IoT platform.

The current version of the Reference Application uses: Mbed OS v5.11.0 and Mbed Cloud Client v2.1.1.

Note: references to Mbed Cloud and Pelion Device Managament are interchangeable.  For example: “Mbed Cloud Client enables the device to communicate with Pelion Device Management” and “Mbed Cloud Client enables the device to communicate with Mbed Cloud” have the same meaning.  In later versions we will remove references to Mbed Cloud inline with branding guidelines.

# Pre-requisites:
1. Hardware:
    * RD2541 based board or platform (the application was developed and tested using the Example Implemenation Hardware - MCB/MTB
    * A micro USB cable
    * An NB-IoT SIM Card

2. Software:
    * mbed CLI - https://github.com/ARMmbed/mbed-cli
    * Pelion account for API keys & developer certificate.

# Getting started:

1. Import this example to your local machine with mbed CLI:
    ``` mbed import https://github.com/ARMmbed/reference-design-RD2541.git ```

2. Move into the reference-design-RD2541 directory ``` cd reference-design-RD2541 ``` . Run the command ``` mbed deploy ``` to get all the sources downloaded & added to the current directory.
    * Login to your Pelion Cloud account on a browser & follow the steps below:
        * Navigate to Device identity > Certificates
        * Select the certificate created by your account admin and click on "Download Developer C file"
        * Save the file mbed_cloud_dev_credentials.c to the location of the reference-design-RD2541 directory. You have to overwrite the contents of the existing file there.

3. Select UDP (default) or TCP to be used by Pelion Cloud Client in mbed_app.json.
    * For UDP set ```"mcc_transport_mode"                : 2```
    * For TCP set ```"mcc_transport_mode"                : 0```

4.  Run the command ``` mbed compile -t GCC_ARM -m MTB_ADV_WISE_1570 ``` to compile the application. The application has been tested to work with all three compilers GCC_ARM, ARM, IAR.

5. Flash the compiled binary (./BUILD/MTB_ADV_WISE_1570/ARM/reference-design-RD2541.hex) to your device. Open a serial terminal at 115200, 8-N-1 & observe the output.

6. The device starts the example, connects to network & after a few moments connects & registers to Pelion Cloud. It then prints out a device ID. Make a note of this.

* You can now check your simulated button counter's value in the Pelion portal as well.
* Navigate to "Device Directory" in the portal and click on your device's device ID (step 5). This will open up a new drawer on the page.
* Scroll down to LWM2M resource "Button_count" and click on it.
* A new window opens and you will be able to see the counter & a graph increment every time the resource value increments.

7. Congratulations, your device is now connected to Pelion IoT platform. Please proceed to the FW update section now.

# Demonstrate a remote Firmware update:

1. Now you need to setup Mbed OS on the device to receive FW updates:
	* Login to your Pelion Cloud portal.
	* Create an API device key (Access Management -> API Keys -> Create a new API key)
	* Ensure you copy the API key generated as this is visible only once.

2. Install the `CLOUD_SDK_API_KEY`

   `mbed config -G CLOUD_SDK_API_KEY ak_1MDE1...<snip>`

3. Initialize the certificates for your application:

   ``` mbed dm init -d "<your company name in Pelion DM>" --model-name "<product-model-identifier>" -q --force ```

   The domain name must have a ".com" as a mandatory requirement and the model name can be an string of your choice.

### Note the use of --force as this option overrides the defaults provided in the application. This is a mandatory parameter.

## IMPORTANT: You must initialize the certificates before you flash the device as these certificates need to be embedded within the device in order for it to receive remote updates.

4. With the certificates initialized, there will now be a ".update-certificates" directory created within the application's root directory.

5. Now, compile the application:
``` mbed compile -t GCC_ARM -m MTB_ADV_WISE_1570 ```

6. Program your device with the ``` ./BUILD/MTB_ADV_WISE_1570/ARM/reference-design-RD2541.hex ``` just generated. You can either drag-drop via the GUI of your OS or you can use the corresponding copy command from the terminal for your OS.

7. Open a serial terminal application such as PuTTY and observe the output. You will notice the device boots up, connects to the network, starts Pelion Cloud Client and registers to the Cloud.

8. Once the device registers, make a note of the device ID assigned to the device.

9. Modify the main.cpp in the source directory to add a small printf statement - this is already provided in the code. So, you can un-comment this line in order to demo a FW update.

10. Use the mbed tool to perform a remote update on the device:

   ``` mbed dm update device -D <device ID> -t GCC_ARM -m MTB_ADV_WISE_1570 ```

   where device ID is the one obtained from step 8.

### Note for UDP the update campaign needs to be started in 30 second window after registering. In case you miss this window, just reboot the device, wait for it to register + 30 seconds, and the update should start. This is required because with UDP the only way to start  update campaign is to forcibly trigger re-registration to cloud. This is implemented in the main.cpp and only used when UDP is enabled.

11. You will notice from the serial terminal that the device gets a request to update the FW, then this is authorized, then the download of the new FW starts, finishes and the bootloader verifies the authencity of the new FW, installs it on the device and reboots.

12. You will now notice the new printf statement appear on the device logs which indicates that the newly built application binary is now installed and running on the device i.e. the device has been remotely updated.

   The success logs should look like the below on a serial terminal, although your device *will* be assigned a different device ID and will have a different IP address.

```
..\Modules\Reference_Apps\RD2541\mbed-cloud-example>mbed sterm -b 115200 -r

[BOOT] Mbed Bootloader
[BOOT] ARM: 00000000000000000000
[BOOT] OEM: 00000000000000000000
[BOOT] Layout: 0 8007698
[BOOT] Active firmware integrity check:
[BOOT] SHA256: 56FCB0439189880EC5CC87B720ACA229995EF65743A0CD11329A6E1ED7C14C44
[BOOT] Version: 1548437876
[BOOT] Slot 0 is empty
[BOOT] Active firmware up-to-date
[BOOT] Application's start address: 0x8008400
[BOOT] Application's jump address: 0x802E3C5
[BOOT] Application's stack address: 0x10008000
[BOOT] Forwarding to application...

Starting Simple Pelion Device Management Client example
Connecting to the network...
Connected to the network successfully. IP address: 10.41.80.14
Initialized Pelion Client. Registering...
Simulated button clicked 1 times
Simulated button clicked 2 times
Simulated button clicked 3 times
Simulated button clicked 4 times
Connected to Pelion Device Management. Endpoint Name: 01688619496300000000000100100197
Simulated button clicked 5 times
Firmware download requested
Authorization granted
Simulated button clicked 6 times
Simulated button clicked 7 times
Downloading: [-                                                 ] 0 %Simulated button clicked 8 times
Downloading: [|                                                 ] 0 %Simulated button clicked 9 times
Downloading: [/                                                 ] 1 %Simulated button clicked 10 times
Downloading: [-                                                 ] 1 %Simulated button clicked 11 times
Downloading: [\                                                 ] 1 %Simulated button clicked 12 times
Downloading: [+/                                                ] 2 %Simulated button clicked 13 times
Downloading: [+-                                                ] 2 %Simulated button clicked 14 times
Downloading: [+|                                                ] 2 %Simulated button clicked 15 times
Downloading: [+/                                                ] 3 %Simulated button clicked 16 times
Downloading: [+-                                                ] 3 %Simulated button clicked 17 times
Downloading: [+\                                                ] 3 %Simulated button clicked 18 times
Downloading: [++/                                               ] 4 %Simulated button clicked 19 times
Downloading: [++-                                               ] 4 %Simulated button clicked 20 times
Downloading: [++|                                               ] 4 %Simulated button clicked 21 times
Downloading: [++/                                               ] 5 %Simulated button clicked 22 times
Downloading: [++-                                               ] 5 %Simulated button clicked 23 times
Downloading: [++\                                               ] 5 %Simulated button clicked 24 times
Downloading: [+++/                                              ] 6 %Simulated button clicked 25 times
Downloading: [+++-                                              ] 6 %Simulated button clicked 26 times
Downloading: [+++|                                              ] 6 %Simulated button clicked 27 times
Downloading: [+++/                                              ] 7 %Simulated button clicked 28 times
Downloading: [+++-                                              ] 7 %Simulated button clicked 29 times
Downloading: [+++\                                              ] 7 %Simulated button clicked 30 times
Downloading: [++++/                                             ] 8 %Simulated button clicked 31 times
Downloading: [++++\                                             ] 8 %Simulated button clicked 32 times
Downloading: [++++|                                             ] 9 %Simulated button clicked 33 times
Downloading: [++++/                                             ] 9 %Simulated button clicked 34 times
Downloading: [++++-                                             ] 9 %Simulated button clicked 35 times
Downloading: [++++\                                             ] 9 %Simulated button clicked 36 times
Downloading: [+++++/                                            ] 10 %Simulated button clicked 37 times
Downloading: [+++++\                                            ] 10 %Simulated button clicked 38 times
Downloading: [+++++|                                            ] 11 %Simulated button clicked 39 times
Downloading: [+++++/                                            ] 11 %Simulated button clicked 40 times
Downloading: [+++++-                                            ] 11 %Simulated button clicked 41 times
Downloading: [+++++\                                            ] 11 %Simulated button clicked 42 times
Downloading: [++++++/                                           ] 12 %Simulated button clicked 43 times
Downloading: [++++++\                                           ] 12 %Simulated button clicked 44 times
Downloading: [++++++|                                           ] 13 %Simulated button clicked 45 times
Downloading: [++++++/                                           ] 13 %Simulated button clicked 46 times
Downloading: [++++++-                                           ] 13 %Simulated button clicked 47 times
Downloading: [++++++\                                           ] 13 %Simulated button clicked 48 times
Downloading: [+++++++-                                          ] 14 %Simulated button clicked 49 times
Downloading: [+++++++\                                          ] 14 %Simulated button clicked 50 times
Downloading: [+++++++|                                          ] 15 %Simulated button clicked 51 times
Downloading: [+++++++/                                          ] 15 %Simulated button clicked 52 times
Downloading: [+++++++-                                          ] 15 %Simulated button clicked 53 times
Downloading: [++++++++/                                         ] 16 %Simulated button clicked 54 times
Downloading: [++++++++\                       Simulated button clicked 55 times
Downloading: [++++++++|                                         ] 16 %Simulated button clicked 56 times
Downloading: [++++++++/                                         ] 17 %Simulated button clicked 57 times
Downloading: [++++++++-                                         ] 17 %Simulated button clicked 58 times
Downloading: [++++++++\                                         ] 17 %Simulated button clicked 59 times
Downloading: [+++++++++/                                        ] 18 %Simulated button clicked 60 times
Downloading: [+++++++++\                                        ] 18 %Simulated button clicked 61 times
Downloading: [+++++++++|                                        ] 18 %Simulated button clicked 62 times
Downloading: [+++++++++/                                        ] 19 %Simulated button clicked 63 times
Downloading: [+++++++++-                                        ] 19 %Simulated button clicked 64 times
Downloading: [+++++++++\                                        ] 19 %Simulated button clicked 65 times
Downloading: [++++++++++/                                       ] 20 %Simulated button clicked 66 times
Downloading: [++++++++++\                                       ] 20 %Simulated button clicked 67 times
Downloading: [++++++++++|                                       ] 20 %Simulated button clicked 68 times
Downloading: [++++++++++/                                       ] 21 %Simulated button clicked 69 times
Downloading: [++++++++++-                                       ] 21 %Simulated button clicked 70 times
Downloading: [++++++++++\                                       ] 21 %Simulated button clicked 71 times
Downloading: [+++++++++++/                                      ] 22 %Simulated button clicked 72 times
Downloading: [+++++++++++\                                      ] 22 %Simulated button clicked 73 times
Downloading: [+++++++++++|                                      ] 22 %Simulated button clicked 74 times
Downloading: [+++++++++++/                                      ] 23 %Simulated button clicked 75 times
Downloading: [+++++++++++-                                      ] 23 %Simulated button clicked 76 times
Downloading: [+++++++++++\                                      ] 23 %Simulated button clicked 77 times
Downloading: [++++++++++++-                                     ] 24 %Simulated button clicked 78 times
Downloading: [++++++++++++\                                     ] 24 %Simulated button clicked 79 times
Downloading: [++++++++++++|                                     ] 25 %Simulated button clicked 80 times
Downloading: [++++++++++++/                                     ] 25 %Simulated button clicked 81 times
Downloading: [++++++++++++-                                     ] 25 %Simulated button clicked 82 times
Downloading: [++++++++++++\                                     ] 25 %Simulated button clicked 83 times
Downloading: [+++++++++++++/                                    ] 26 %Simulated button clicked 84 times
Downloading: [+++++++++++++\                                    ] 26 %Simulated button clicked 85 times
Downloading: [+++++++++++++|                                    ] 27 %Simulated button clicked 86 times
Downloading: [+++++++++++++/                                    ] 27 %Simulated button clicked 87 times
Downloading: [+++++++++++++-                                    ] 27 %Simulated button clicked 88 times
Downloading: [+++++++++++++\                                    ] 27 %Simulated button clicked 89 times
Downloading: [++++++++++++++/                                   ] 28 %Simulated button clicked 90 times
Downloading: [++++++++++++++\                                   ] 28 %Simulated button clicked 91 times
Downloading: [++++++++++++++|                                   ] 29 %Simulated button clicked 92 times
Downloading: [++++++++++++++/                                   ] 29 %Simulated button clicked 93 times
Downloading: [++++++++++++++-                                   ] 29 %Simulated button clicked 94 times
Downloading: [++++++++++++++\                                   ] 29 %Simulated button clicked 95 times
Downloading: [+++++++++++++++/                                  ] 30 %Simulated button clicked 96 times
Downloading: [+++++++++++++++\                                  ] 30 %Simulated button clicked 97 times
Downloading: [+++++++++++++++|                                  ] 31 %Simulated button clicked 98 times
Downloading: [+++++++++++++++/                                  ] 31 %Simulated button clicked 99 times
Downloading: [+++++++++++++++-                                  ] 31 %Simulated button clicked 100 times
Downloading: [++++++++++++++++/                                 ] 32 %Simulated button clicked 101 times
Downloading: [++++++++++++++++-                                 ] 32 %Simulated button clicked 102 times
Downloading: [++++++++++++++++|                                 ] 32 %Simulated button clicked 103 times
Downloading: [++++++++++++++++/                                 ] 33 %Simulated button clicked 104 times
Downloading: [++++++++++++++++-                                 ] 33 %Simulated button clicked 105 times
Downloading: [++++++++++++++++\                                 ] 33 %Simulated button clicked 106 times
Downloading: [+++++++++++++++++/                                ] 34 %Simulated button clicked 107 times
Downloading: [+++++++++++++++++-                                ] 34 %Simulated button clicked 108 times
Downloading: [+++++++++++++++++|                                ] 34 %Simulated button clicked 109 times
Downloading: [+++++++++++++++++/                                ] 35 %Simulated button clicked 110 times
Simulated button clicked 111 times
Downloading: [+++++++++++++++++\                                ] 35 %Simulated button clicked 112 times
Downloading: [++++++++++++++++++/                               ] 36 %Simulated button clicked 113 times
Downloading: [++++++++++++++++++-                               ] 36 %Simulated button clicked 114 times
Downloading: [++++++++++++++++++\                               ] 36 %Simulated button clicked 115 times
Downloading: [++++++++++++++++++|                               ] 36 %Simulated button clicked 116 times
Downloading: [++++++++++++++++++/                               ] 37 %Simulated button clicked 117 times
Downloading: [++++++++++++++++++\                               ] 37 %Simulated button clicked 118 times
Downloading: [+++++++++++++++++++/                              ] 38 %Simulated button clicked 119 times
Downloading: [+++++++++++++++++++-                              ] 38 %Simulated button clicked 120 times
Downloading: [+++++++++++++++++++\                              ] 38 %Simulated button clicked 121 times
Downloading: [+++++++++++++++++++|                              ] 39 %Simulated button clicked 122 times
Downloading: [+++++++++++++++++++/                              ] 39 %Simulated button clicked 123 times
Downloading: [+++++++++++++++++++\                              ] 39 %Simulated button clicked 124 times
Downloading: [++++++++++++++++++++/                             ] 40 %Simulated button clicked 125 times
Downloading: [++++++++++++++++++++-                             ] 40 %Simulated button clicked 126 times
Downloading: [++++++++++++++++++++\                             ] 40 %Simulated button clicked 127 times
Downloading: [++++++++++++++++++++|                             ] 41 %Simulated button clicked 128 times
Downloading: [++++++++++++++++++++/                             ] 41 %Simulated button clicked 129 times
Downloading: [++++++++++++++++++++\                             ] 41 %Simulated button clicked 130 times
Downloading: [+++++++++++++++++++++/                            ] 42 %Simulated button clicked 131 times
Downloading: [+++++++++++++++++++++-                            ] 42 %Simulated button clicked 132 times
Downloading: [+++++++++++++++++++++\                            ] 42 %Simulated button clicked 133 times
Downloading: [+++++++++++++++++++++|                            ] 43 %Simulated button clicked 134 times
Downloading: [+++++++++++++++++++++-                            ] 43 %Simulated button clicked 135 times
Downloading: [+++++++++++++++++++++\                            ] 43 %Simulated button clicked 136 times
Downloading: [++++++++++++++++++++++/                           ] 44 %Simulated button clicked 137 times
Downloading: [++++++++++++++++++++++-                           ] 44 %Simulated button clicked 138 times
Downloading: [++++++++++++++++++++++\                           ] 44 %Simulated button clicked 139 times
Downloading: [++++++++++++++++++++++|                           ] 45 %Simulated button clicked 140 times
Downloading: [++++++++++++++++++++++-                           ] 45 %Simulated button clicked 141 times
Downloading: [++++++++++++++++++++++\                           ] 45 %Simulated button clicked 142 times
Downloading: [+++++++++++++++++++++++/                          ] 46 %Simulated button clicked 143 times
Downloading: [+++++++++++++++++++++++-                          ] 46 %Simulated button clicked 144 times
Downloading: [+++++++++++++++++++++++\                          ] 46 %Simulated button clicked 145 times
Downloading: [+++++++++++++++++++++++|                          ] 47 %Simulated button clicked 146 times
Downloading: [+++++++++++++++++++++++-                          ] 47 %Simulated button clicked 147 times
Downloading: [++++++++++++++++++++++++/                         ] 48 %Simulated button clicked 148 times
Downloading: [++++++++++++++++++++++++-                         ] 48 %Simulated button clicked 149 times
Downloading: [++++++++++++++++++++++++\                         ] 48 %Simulated button clicked 150 times
Downloading: [++++++++++++++++++++++++|                         ] 48 %Simulated button clicked 151 times
Downloading: [++++++++++++++++++++++++/                         ] 49 %Simulated button clicked 152 times
Downloading: [++++++++++++++++++++++++\                         ] 49 %Simulated button clicked 153 times
Downloading: [+++++++++++++++++++++++++/                        ] 50 %Simulated button clicked 154 times
Downloading: [+++++++++++++++++++++++++-                        ] 50 %Simulated button clicked 155 times
Downloading: [+++++++++++++++++++++++++\                        ] 50 %Simulated button clicked 156 times
Downloading: [+++++++++++++++++++++++++|                        ] 50 %Simulated button clicked 157 times
Downloading: [+++++++++++++++++++++++++/                        ] 51 %Simulated button clicked 158 times
Downloading: [+++++++++++++++++++++++++\                        ] 51 %Simulated button clicked 159 times
Downloading: [++++++++++++++++++++++++++/                       ] 52 %Simulated button clicked 160 times
Downloading: [++++++++++++++++++++++++++-                       ] 52 %Simulated button clicked 161 times
Downloading: [++++++++++++++++++++++++++\                       ] 52 %Simulated button clicked 162 times
Downloading: [++++++++++++++++++++++++++|                       ] 52 %Simulated button clicked 163 times
Downloading: [++++++++++++++++++++++++++/                       ] 53 %Simulated button clicked 164 times
Downloading: [++++++++++++++++++++++++++\                       ] 53 %Simulated button clicked 165 times
Downloading: [+++++++++++++++++++++++++++/                      ] 54 %Simulated button clicked 166 times
Downloading: [+++++++++++++++++++++++++++-                      ] 54 %Simulated button clicked 167 times
Downloading: [+++++++++++++++++++++++++++\                      ] 54 %Simulated button clicked 168 times
Downloading: [+++++++++++++++++++++++++++|                      ] 55 %Simulated button clicked 169 times
Downloading: [+++++++++++++++++++++++++++-                      ] 55 %Simulated button clicked 170 times
Downloading: [+++++++++++++++++++++++++++\                      ] 55 %Simulated button clicked 171 times
Downloading: [++++++++++++++++++++++++++++/                     ] 56 %Simulated button clicked 172 times
Downloading: [++++++++++++++++++++++++++++-                     ] 56 %Simulated button clicked 173 times
Downloading: [++++++++++++++++++++++++++++\                     ] 56 %Simulated button clicked 174 times
Downloading: [++++++++++++++++++++++++++++|                     ] 57 %Simulated button clicked 175 times
Downloading: [++++++++++++++++++++++++++++-                     ] 57 %Simulated button clicked 176 times
Downloading: [++++++++++++++++++++++++++++\                     ] 57 %Simulated button clicked 177 times
Downloading: [+++++++++++++++++++++++++++++/                    ] 58 %Simulated button clicked 178 times
Downloading: [+++++++++++++++++++++++++++++-                    ] 58 %Simulated button clicked 179 times
Downloading: [+++++++++++++++++++++++++++++\                    ] 58 %Simulated button clicked 180 times
Downloading: [+++++++++++++++++++++++++++++|                    ] 59 %Simulated button clicked 181 times
Downloading: [+++++++++++++++++++++++++++++-                    ] 59 %Simulated button clicked 182 times
Downloading: [+++++++++++++++++++++++++++++\                    ] 59 %Simulated button clicked 183 times
Downloading: [++++++++++++++++++++++++++++++/                   ] 60 %Simulated button clicked 184 times
Downloading: [++++++++++++++++++++++++++++++-                   ] 60 %Simulated button clicked 185 times
Downloading: [++++++++++++++++++++++++++++++\                   ] 60 %Simulated button clicked 186 times
Downloading: [++++++++++++++++++++++++++++++|                   ] 61 %Simulated button clicked 187 times
Downloading: [++++++++++++++++++++++++++++++-                   ] 61 %Simulated button clicked 188 times
Downloading: [+++++++++++++++++++++++++++++++/                  ] 62 %Simulated button clicked 189 times
Downloading: [+++++++++++++++++++++++++++++++-                  ] 62 %Simulated button clicked 190 times
Downloading: [+++++++++++++++++++++++++++++++\                  ] 62 %Simulated button clicked 191 times
Downloading: [+++++++++++++++++++++++++++++++|                  ] 62 %Simulated button clicked 192 times
Downloading: [+++++++++++++++++++++++++++++++/                  ] 63 %Simulated button clicked 193 times
Downloading: [+++++++++++++++++++++++++++++++\                  ] 63 %Simulated button clicked 194 times
Downloading: [++++++++++++++++++++++++++++++++/                 ] 64 %Simulated button clicked 195 times
Downloading: [++++++++++++++++++++++++++++++++-                 ] 64 %Simulated button clicked 196 times
Downloading: [++++++++++++++++++++++++++++++++\                 ] 64 %Simulated button clicked 197 times
Downloading: [++++++++++++++++++++++++++++++++|                 ] 64 %Simulated button clicked 198 times
Downloading: [++++++++++++++++++++++++++++++++-                 ] 65 %Simulated button clicked 199 times
Downloading: [++++++++++++++++++++++++++++++++\                 ] 65 %Simulated button clicked 200 times
Downloading: [+++++++++++++++++++++++++++++++++/                ] 66 %Simulated button clicked 201 times
Downloading: [+++++++++++++++++++++++++++++++++-                ] 66 %Simulated button clicked 202 times
Downloading: [+++++++++++++++++++++++++++++++++\                ] 66 %Simulated button clicked 203 times
Downloading: [+++++++++++++++++++++++++++++++++|                ] 66 %Simulated button clicked 204 times
Downloading: [+++++++++++++++++++++++++++++++++-                ] 67 %Simulated button clicked 205 times
Downloading: [+++++++++++++++++++++++++++++++++\                ] 67 %Simulated button clicked 206 times
Downloading: [++++++++++++++++++++++++++++++++++/               ] 68 %Simulated button clicked 207 times
Downloading: [++++++++++++++++++++++++++++++++++-               ] 68 %Simulated button clicked 208 times
Downloading: [++++++++++++++++++++++++++++++++++\               ] 68 %Simulated button clicked 209 times
Downloading: [++++++++++++++++++++++++++++++++++|               ] 68 %Simulated button clicked 210 times
Downloading: [++++++++++++++++++++++++++++++++++-               ] 69 %Simulated button clicked 211 times
Downloading: [++++++++++++++++++++++++++++++++++\               ] 69 %Simulated button clicked 212 times
Downloading: [+++++++++++++++++++++++++++++++++++/              ] 70 %Simulated button clicked 213 times
Downloading: [+++++++++++++++++++++++++++++++++++-              ] 70 %Simulated button clicked 214 times
Downloading: [+++++++++++++++++++++++++++++++++++\              ] 70 %Simulated button clicked 215 times
Downloading: [+++++++++++++++++++++++++++++++++++|              ] 71 %Simulated button clicked 216 times
Downloading: [+++++++++++++++++++++++++++++++++++-              ] 71 %Simulated button clicked 217 times
Downloading: [+++++++++++++++++++++++++++++++++++\              ] 71 %Simulated button clicked 218 times
Downloading: [++++++++++++++++++++++++++++++++++++/             ] 72 %Simulated button clicked 219 times
Downloading: [++++++++++++++++++++++++++++++++++++-             ] 72 %Simulated button clicked 220 times
Downloading: [++++++++++++++++++++++++++++++++++++\             ] 72 %Simulated button clicked 221 times
Downloading: [++++++++++++++++++++++++++++++++++++|             ] 73 %Simulated button clicked 222 times
Downloading: [++++++++++++++++++++++++++++++++++++-             ] 73 %Simulated button clicked 223 times
Downloading: [++++++++++++++++++++++++++++++++++++\             ] 73 %Simulated button clicked 224 times
Downloading: [+++++++++++++++++++++++++++++++++++++/            ] 74 %Simulated button clicked 225 times
Downloading: [+++++++++++++++++++++++++++++++++++++-            ] 74 %Simulated button clicked 226 times
Downloading: [+++++++++++++++++++++++++++++++++++++\            ] 74 %Simulated button clicked 227 times
Downloading: [+++++++++++++++++++++++++++++++++++++|            ] 75 %Simulated button clicked 228 times
Downloading: [+++++++++++++++++++++++++++++++++++++-            ] 75 %Simulated button clicked 229 times
Downloading: [+++++++++++++++++++++++++++++++++++++\            ] 75 %Simulated button clicked 230 times
Downloading: [++++++++++++++++++++++++++++++++++++++/           ] 76 %Simulated button clicked 231 times
Downloading: [++++++++++++++++++++++++++++++++++++++-           ] 76 %Simulated button clicked 232 times
Downloading: [++++++++++++++++++++++++++++++++++++++\           ] 76 %Simulated button clicked 233 times
Downloading: [++++++++++++++++++++++++++++++++++++++/           ] 77 %Simulated button clicked 234 times
Downloading: [++++++++++++++++++++++++++++++++++++++-           ] 77 %Simulated button clicked 235 times
Downloading: [+++++++++++++++++++++++++++++++++++++++/          ] 78 %Simulated button clicked 236 times
Downloading: [+++++++++++++++++++++++++++++++++++++++-          ] 78 %Simulated button clicked 237 times
Downloading: [+++++++++++++++++++++++++++++++++++++++\          ] 78 %Simulated button clicked 238 times
Downloading: [+++++++++++++++++++++++++++++++++++++++|          ] 78 %Simulated button clicked 239 times
Downloading: [+++++++++++++++++++++++++++++++++++++++-          ] 79 %Simulated button clicked 240 times
Downloading: [+++++++++++++++++++++++++++++++++++++++\          ] 79 %Simulated button clicked 241 times
Downloading: [++++++++++++++++++++++++++++++++++++++++/         ] 80 %Simulated button clicked 242 times
Downloading: [++++++++++++++++++++++++++++++++++++++++-         ] 80 %Simulated button clicked 243 times
Downloading: [++++++++++++++++++++++++++++++++++++++++\         ] 80 %Simulated button clicked 244 times
Downloading: [++++++++++++++++++++++++++++++++++++++++|         ] 80 %Simulated button clicked 245 times
Downloading: [++++++++++++++++++++++++++++++++++++++++-         ] 81 %Simulated button clicked 246 times
Downloading: [++++++++++++++++++++++++++++++++++++++++\         ] 81 %Simulated button clicked 247 times
Downloading: [+++++++++++++++++++++++++++++++++++++++++/        ] 82 %Simulated button clicked 248 times
Downloading: [+++++++++++++++++++++++++++++++++++++++++-        ] 82 %Simulated button clicked 249 times
Downloading: [+++++++++++++++++++++++++++++++++++++++++\        ] 82 %Simulated button clicked 250 times
Downloading: [+++++++++++++++++++++++++++++++++++++++++|        ] 82 %Simulated button clicked 251 times
Downloading: [+++++++++++++++++++++++++++++++++++++++++-        ] 83 %Simulated button clicked 252 times
Downloading: [+++++++++++++++++++++++++++++++++++++++++\        ] 83 %Simulated button clicked 253 times
Downloading: [++++++++++++++++++++++++++++++++++++++++++/       ] 84 %Simulated button clicked 254 times
Downloading: [++++++++++++++++++++++++++++++++++++++++++-       ] 84 %Simulated button clicked 255 times
Downloading: [++++++++++++++++++++++++++++++++++++++++++\       ] 84 %Simulated button clicked 256 times
Downloading: [++++++++++++++++++++++++++++++++++++++++++|       ] 85 %Simulated button clicked 257 times
Downloading: [++++++++++++++++++++++++++++++++++++++++++-       ] 85 %Simulated button clicked 258 times
Downloading: [++++++++++++++++++++++++++++++++++++++++++\       ] 85 %Simulated button clicked 259 times
Downloading: [+++++++++++++++++++++++++++++++++++++++++++/      ] 86 %Simulated button clicked 260 times
Downloading: [+++++++++++++++++++++++++++++++++++++++++++-      ] 86 %Simulated button clicked 261 times
Downloading: [+++++++++++++++++++++++++++++++++++++++++++\      ] 86 %Simulated button clicked 262 times
Downloading: [+++++++++++++++++++++++++++++++++++++++++++/      ] 87 %Simulated button clicked 263 times
Downloading: [+++++++++++++++++++++++++++++++++++++++++++-      ] 87 %Simulated button clicked 264 times
Downloading: [+++++++++++++++++++++++++++++++++++++++++++\      ] 87 %Simulated button clicked 265 times
Downloading: [++++++++++++++++++++++++++++++++++++++++++++/     ] 88 %Simulated button clicked 266 times
Downloading: [++++++++++++++++++++++++++++++++++++++++++++-     ] 88 %Simulated button clicked 267 times
Downloading: [++++++++++++++++++++++++++++++++++++++++++++\     ] 88 %Simulated button clicked 268 times
Downloading: [++++++++++++++++++++++++++++++++++++++++++++/     ] 89 %Simulated button clicked 269 times
Downloading: [++++++++++++++++++++++++++++++++++++++++++++-     ] 89 %Simulated button clicked 270 times
Downloading: [++++++++++++++++++++++++++++++++++++++++++++\     ] 89 %Simulated button clicked 271 times
Downloading: [+++++++++++++++++++++++++++++++++++++++++++++/    ] 90 %Simulated button clicked 272 times
Downloading: [+++++++++++++++++++++++++++++++++++++++++++++-    ] 90 %Simulated button clicked 273 times
Downloading: [+++++++++++++++++++++++++++++++++++++++++++++\    ] 90 %Simulated button clicked 274 times
Downloading: [+++++++++++++++++++++++++++++++++++++++++++++/    ] 91 %Simulated button clicked 275 times
Downloading: [+++++++++++++++++++++++++++++++++++++++++++++-    ] 91 %Simulated button clicked 276 times
Downloading: [+++++++++++++++++++++++++++++++++++++++++++++\    ] 91 %Simulated button clicked 277 times
Downloading: [++++++++++++++++++++++++++++++++++++++++++++++/   ] 92 %Simulated button clicked 278 times
Downloading: [++++++++++++++++++++++++++++++++++++++++++++++-   ] 92 %Simulated button clicked 279 times
Downloading: [++++++++++++++++++++++++++++++++++++++++++++++\   ] 92 %Simulated button clicked 280 times
Downloading: [++++++++++++++++++++++++++++++++++++++++++++++/   ] 93 %Simulated button clicked 281 times
Downloading: [++++++++++++++++++++++++++++++++++++++++++++++-   ] 93 %Simulated button clicked 282 times
Downloading: [+++++++++++++++++++++++++++++++++++++++++++++++/  ] 94 %Simulated button clicked 283 times
Downloading: [+++++++++++++++++++++++++++++++++++++++++++++++-  ] 94 %Simulated button clicked 284 times
Downloading: [+++++++++++++++++++++++++++++++++++++++++++++++\  ] 94 %Simulated button clicked 285 times
Downloading: [+++++++++++++++++++++++++++++++++++++++++++++++|  ] 94 %Simulated button clicked 286 times
Downloading: [+++++++++++++++++++++++++++++++++++++++++++++++-  ] 95 %Simulated button clicked 287 times
Downloading: [+++++++++++++++++++++++++++++++++++++++++++++++\  ] 95 %Simulated button clicked 288 times
Downloading: [++++++++++++++++++++++++++++++++++++++++++++++++/ ] 96 %Simulated button clicked 289 times
Downloading: [++++++++++++++++++++++++++++++++++++++++++++++++- ] 96 %Simulated button clicked 290 times
Downloading: [++++++++++++++++++++++++++++++++++++++++++++++++\ ] 96 %Simulated button clicked 291 times
Downloading: [++++++++++++++++++++++++++++++++++++++++++++++++| ] 96 %Simulated button clicked 292 times
Downloading: [++++++++++++++++++++++++++++++++++++++++++++++++- ] 97 %Simulated button clicked 293 times
Downloading: [++++++++++++++++++++++++++++++++++++++++++++++++\ ] 97 %Simulated button clicked 294 times
Downloading: [+++++++++++++++++++++++++++++++++++++++++++++++++/] 98 %Simulated button clicked 295 times
Downloading: [+++++++++++++++++++++++++++++++++++++++++++++++++-] 98 %Simulated button clicked 296 times
Downloading: [+++++++++++++++++++++++++++++++++++++++++++++++++\] 98 %Simulated button clicked 297 times
Downloading: [+++++++++++++++++++++++++++++++++++++++++++++++++|] 98 %Simulated button clicked 298 times
Downloading: [+++++++++++++++++++++++++++++++++++++++++++++++++-] 99 %Simulated button clicked 299 times
Downloading: [+++++++++++++++++++++++++++++++++++++++++++++++++\] 99 %Simulated button clicked 300 times
Downloading: [++++++++++++++++++++++++++++++++++++++++++++++++++] 100 %
Download completed
Simulated button clicked 301 times
Firmware install requested
Authorization granted
[BOOT] Mbed Bootloader
[BOOT] ARM: 00000000000000000000
[BOOT] OEM: 00000000000000000000
[BOOT] Layout: 0 8007698
[BOOT] Active firmware integrity check:
[BOOT] SHA256: 56FCB0439189880EC5CC87B720ACA229995EF65743A0CD11329A6E1ED7C14C44
[BOOT] Version: 1548437876
[BOOT] Slot 0 firmware integrity check:
[BOOT] SHA256: 92575E40B763460035B9137D56619184B09F2A998AC464A7A2A5B254C0E17102
[BOOT] Version: 1548438120
[BOOT] Update active firmware using slot 0:
[BOOT] Verify new active firmware:
[BOOT] New active firmware is valid
[BOOT] Application's start address: 0x8008400
[BOOT] Application's jump address: 0x805DD85
[BOOT] Application's stack address: 0x10007F88
[BOOT] Forwarding to application...
ÿStarting Simple Pelion Device Management Client example
FW updated OTA
Connecting to the network...
Connected to the network successfully. IP address: 10.41.80.19
Initialized Pelion Client. Registering...
Simulated button clicked 1 times
Simulated button clicked 2 times
Simulated button clicked 3 times
Simulated button clicked 4 times
Connected to Pelion Device Management. Endpoint Name: 01688619496300000000000100100197
Simulated button clicked 5 times
Simulated button clicked 6 times
Simulated button clicked 7 times
Simulated button clicked 8 times
Simulated button clicked 9 times
Simulated button clicked 10 times
Simulated button clicked 11 times
Simulated button clicked 12 times

```

## Also note that the device ID remains the same after the FW update. This indicates that the SOTP regions were not over-written while perfoming the update.

# Flash and static RAM usage
Default profile, GCC_ARM:
Total Static RAM memory (data + bss): 30752(+30752) bytes
Total Flash memory (text + data): 414014(+414014) bytes

Default profile, ARM:
Total Static RAM memory (data + bss): 28742(+28742) bytes
Total Flash memory (text + data): 413517(+413517) bytes

Default profile, IAR:
Total Static RAM memory (data + bss): 28665(+28665) bytes
Total Flash memory (text + data): 356185(+356185) bytes

Release profile, GCC_ARM:
Total Static RAM memory (data + bss): 30752(+30752) bytes
Total Flash memory (text + data): 396433(+396433) bytes

Release profile, ARM:
Total Static RAM memory (data + bss): 28742(+28742) bytes
Total Flash memory (text + data): 348980(+348980) bytes

Release profile, IAR:
Total Static RAM memory (data + bss): 28658(+28658) bytes
Total Flash memory (text + data): 323198(+323198) bytes

# Dynamic RAM usage (approximate, measurement affects results)

Debug profile, GCC_ARM:
Unable to measure. Out of memory due to measurement mode overhead.

Debug profile, ARM:
Unable to measure. Out of memory due to measurement mode overhead.

Debug profile, IAR:
Unable to measure. Out of memory due to measurement mode overhead.

Release profile (max used / available), GCC_ARM:
56525 / 67552

Release profile (max used / available), ARM:
54613 / 68384

Release profile (max used / available), IAR:
52901 / 65536
