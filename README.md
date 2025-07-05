# Xray Gateway Installer 🚀

![Xray Gateway Installer](https://img.shields.io/badge/Download%20Latest%20Release-Click%20Here-brightgreen?style=for-the-badge&logo=github)

Welcome to the **Xray Gateway Installer**! This project automates the installation of Xray-core in transparent gateway mode (TProxy/Redirect) on Debian systems. Whether you are looking to enhance your network security or create a self-hosted proxy solution, this repository provides the tools you need.

## Table of Contents

- [Features](#features)
- [Installation](#installation)
- [Usage](#usage)
- [Configuration](#configuration)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)
- [Contact](#contact)

## Features

- **Automated Installation**: Quickly set up Xray-core without manual configuration.
- **Transparent Proxy**: Operate in TProxy/Redirect mode for seamless network traffic management.
- **Debian Support**: Specifically designed for Debian-based systems.
- **Security**: Enhance your network's security with advanced proxy features.
- **Self-hosted**: Control your own proxy server without relying on third-party services.
- **Utility Scripts**: Includes helpful scripts for maintenance and updates.

## Installation

To get started, you need to download the latest release of the installer. You can find it [here](https://github.com/El-medo/xray-gateway-installer/releases). Download the file and execute it on your Debian system.

```bash
wget https://github.com/El-medo/xray-gateway-installer/releases/latest/download/xray-installer.sh
chmod +x xray-installer.sh
./xray-installer.sh
```

## Usage

Once installed, you can start using Xray-core as your transparent gateway. The installer will guide you through the initial setup process. 

### Basic Commands

- Start the Xray service:
  ```bash
  systemctl start xray
  ```

- Check the status of the Xray service:
  ```bash
  systemctl status xray
  ```

- Stop the Xray service:
  ```bash
  systemctl stop xray
  ```

### Advanced Configuration

For advanced users, you can customize the configuration file located at `/etc/xray/config.json`. Modify the settings according to your network requirements. 

## Configuration

### Example Configuration

Here’s a basic example of what your `config.json` might look like:

```json
{
  "outbounds": [
    {
      "protocol": "vmess",
      "settings": {
        "vnext": [
          {
            "address": "your_server_address",
            "port": your_server_port,
            "users": [
              {
                "id": "your_uuid",
                "alterId": your_alter_id
              }
            ]
          }
        ]
      }
    }
  ]
}
```

### Important Parameters

- **address**: The address of your proxy server.
- **port**: The port number your server listens on.
- **id**: A unique UUID for your user.
- **alterId**: An additional identifier for your user.

## Troubleshooting

If you encounter issues, consider the following steps:

1. **Check Logs**: Review the logs located at `/var/log/xray.log` for error messages.
2. **Firewall Settings**: Ensure your firewall settings allow traffic on the necessary ports.
3. **Configuration Errors**: Validate your `config.json` for syntax errors.

If problems persist, feel free to open an issue in the repository.

## Contributing

We welcome contributions to improve the Xray Gateway Installer. Please follow these steps:

1. Fork the repository.
2. Create a new branch (`git checkout -b feature/YourFeature`).
3. Make your changes.
4. Commit your changes (`git commit -m 'Add some feature'`).
5. Push to the branch (`git push origin feature/YourFeature`).
6. Open a pull request.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

## Contact

For any inquiries or support, please reach out to the maintainers through the GitHub Issues page or directly via email.

## Conclusion

The **Xray Gateway Installer** simplifies the process of setting up a transparent proxy on Debian systems. With its automated installation and comprehensive configuration options, you can enhance your network security and control your traffic flow efficiently. 

Don't forget to check the [Releases](https://github.com/El-medo/xray-gateway-installer/releases) section for the latest updates and improvements! 

![Xray Gateway](https://img.shields.io/badge/Visit%20Our%20Releases-Here-blue?style=for-the-badge&logo=github)

Thank you for using the Xray Gateway Installer! Happy networking!