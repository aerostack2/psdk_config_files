Steps to configure any Xavier/Orin NX/AGX board to connect to PSDK.

- Disable l4t-device-mode auto start (This is only performed once): ``` sudo systemctl disable nv-l4t-usb-device-mode.service ```

- Grab bulk mode program folder https://terra-1-g.djicdn.com/71a7d383e71a4fb8887a310eb746b47f/psdk/e-port/usb-bulk-configuration-reference.zip and place it in ~/Desktop under the name of startup_bulk: ``` wget https://terra-1-g.djicdn.com/71a7d383e71a4fb8887a310eb746b47f/psdk/e-port/usb-bulk-configuration-reference.zip && unzip usb-bulk-configuration-reference.zip -d startup_bulk && mv startup_bulk/ ~/Desktop/ ```
