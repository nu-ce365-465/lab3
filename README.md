# lab3
# Run Azure IoT Edge on the DE10-Nano

This is part 1 of a 4-part series that shows you how to manage the DE10-Nano with Azure IoT Edge and use container-based virtualization to reprogram the onboard FPGA from the Azure Cloud. 

## Table of Contents

- [Run Azure IoT Edge on the DE10-Nano](#run-azure-iot-edge-on-the-de10-nano)
  - [Table of Contents](#table-of-contents)
  - [About this Tutorial](#about-this-tutorial)
    - [Audience](#audience)
    - [Objectives](#objectives)
    - [Prerequisites](#prerequisites)
      - [Optional](#optional)
  - [Step 1: Prepare the DE10-Nano](#step-1-prepare-the-de10-nano)
    - [Download the Image from Terasic](#download-the-image-from-terasic)
    - [Write the Image to a microSD Card](#write-the-image-to-a-microsd-card)
    - [Set up MSEL Switches](#set-up-msel-switches)
  - [Step 2: Power on and Log in](#step-2-power-on-and-log-in)
    - [Serial Console](#serial-console)
    - [SSH](#ssh)
      - [Set up a Static IP Address (Optional)](#set-up-a-static-ip-address-optional)
    - [GUI](#gui)
  - [Step 3: Install Azure IoT Edge](#step-3-install-azure-iot-edge)
    - [Set up apt Package Setting](#set-up-apt-package-setting)
    - [Install Azure IoT Edge Runtime](#install-azure-iot-edge-runtime)
  - [Step 4: Create an IoT Hub Instance on Azure Portal](#step-4-create-an-iot-hub-instance-on-azure-portal)
    - [Register DE10-Nano as an IoT Edge Device](#register-de10-nano-as-an-iot-edge-device)
  - [Step 5: Update the IoT Edge Daemon Config File](#step-5-update-the-iot-edge-daemon-config-file)
  - [Step 6: Set up DNS and Log Policy](#step-6-set-up-dns-and-log-policy)
    - [Set up DNS on Container Runtime](#set-up-dns-on-container-runtime)
  - [Configuration checks (aziot-identity-service)](#configuration-checks-aziot-identity-service)
  - [Connectivity checks (aziot-identity-service)](#connectivity-checks-aziot-identity-service)
  - [Configuration checks](#configuration-checks)
  - [Connectivity checks](#connectivity-checks)
    - [Set up Log Policy](#set-up-log-policy)
  - [Step 7: Expand microSD Card Storage Size](#step-7-expand-microsd-card-storage-size)
  - [Step 8: Deploy a Simulated Temperature Sensor](#step-8-deploy-a-simulated-temperature-sensor)
    - [Check Container Data from the DE10-Nano Device](#check-container-data-from-the-de10-nano-device)
    - [Check Container Data from the Cloud](#check-container-data-from-the-cloud)
      - [Install Azure CLI](#install-azure-cli)
      - [Confirm DE10-Nano Messages with Azure CLI](#confirm-de10-nano-messages-with-azure-cli)
  - [Next Steps](#next-steps)

## About this Tutorial

This tutorial covers the basic hardware and software set up for connecting the Intel(R) Cyclone(R) V SoC on the DE10-Nano to Microsoft\* Azure IoT edge. After completing this tutorial, you can use Azure IoT Edge to manage the DE10-Nano along with other IoT devices in the cloud.

### Audience

Software engineers who need a starting point to connect Intel(R) Cyclone(R) V SoC to Microsoft Azure IoT as an IoT gateway function.

### Objectives

In this tutorial, you will learn how to:

- Prepare the DE10-Nano for use in Azure IoT Edge
- Install Azure IoT Edge runtime on the DE10-Nano
- Deploy a simulated temperature sensor to test Azure IoT Edge
- Verify that the container works from the DE10-Nano and the cloud

### Prerequisites

* Microsoft Azure account
* Development PC with Linux\* Ubuntu16.04 (recommended)
* Development PC must operate on 64-bit architecture & contain 10GB free space for the Modules, as well as 40GB additional free space for Quartus(R) Installation
* Serial console application such as [PuTTY](https://www.putty.org/ ), screen, or minicom on your development PC. 
* DE10-Nano Development Kit
  You can purchase the kit directly from [Terasic][LINK_Terasic_DE10-Nano-Purchase] or from your local Intel distributor.
* microSD to SD memory card adapter and SD card reader if your system does not have a microSD card slot or SD card reader.
  **Note**: The kit comes with a 4GB microSD card. For better performance, you can purchase a card with a minimum of 8GB.

* Ethernet cable  
  
It is helpful but not required to have experience with:
-  [DE10-Nano User Manual][LINK_Terasic_DE10_NANO_User_Manual]
-  [DE10-Nano Getting Started Guide][LINK_Terasic_getting_started_guide]
-  Linux
-  Container technology such as Docker\*

#### Optional

Hardware accessories for enabling the GUI:

* HDMI cable and compatible display
* USB adapter (for type A to micro-B USB cable)
* USB hub
* USB keyboard
* USB mouse

## Step 1: Prepare the DE10-Nano

In this step, you will prepare the DE10-Nano for container virtualization, enabling you to maintain DE10-Nano applications on a cloud service such as Microsoft Azure\*.

### Download the Image from Terasic

The bootable image from Terasic contains Ubuntu 16.04 with a Lightweight X11 Desktop Environment (or LXDE). You can use the Linux GUI, serial console, or ssh.  

1. Download the [Terasic SD Card Image][LINK_Terasic_bootable] onto your development PC.

2. Extract the downloaded image:
   
   ```
   cd ~/Downloads
   unzip DE10-Nano-Cloud-Native.zip
   ```

### Write the Image to a microSD Card

If your development PC does not have an SD card slot, use a microSD to SD memory card adapter and SD card reader to complete this step.

1. Insert the microSD into your development PC and identify its device path with `lsblk` or `fdisk -l`.  
   
   

   Input:

   ```
   lsblk
   ```

   Output:

   ```
    NAME              MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
    ...
    <Your Device Path>                 8:0    1  <card size>G  0 disk
    ...
   ```

2. Using the `dd` command, write the image to the microSD card.

   ```
   sudo dd if=DE10-Nano-Cloud-Native.img of=/dev/<your device path> status=progress
   ```
**Note**: Take precautions to select the correct path. If you choose another device path, it may crash your development PC. Use `status=progress` to check the progress of the dd command.

3. For more information on the dd command, see the Linux manual (recommended).

   ```
   man dd
   ```

   An excerpt of the manual:

   ```
   dd - convert and copy a file

   if=FILE
         read from FILE instead of stdin

   of=FILE
         write to FILE instead of stdout

   status=LEVEL
           The  LEVEL of information to print to stderr; 
           'none' suppresses everything but error messages,
           'noxfer' suppresses the final transfer statistics,
           'progress' shows periodic transfer statistics
   ```

### Set up MSEL Switches

The MSEL switches on DE10-Nano are used to configure the board. In this step, you will set the MSEL switches to run Linux.  

1. Set  MSEL[4:0] to "01010", according to section 2.2 (MSEL Settings) of the [DE10-Nano Getting Started Guide][LINK_Terasic_getting_started_guide].
   
   ![MSEL-settings](picture/MSEL-settings.png)


## Step 2: Power on and Log in

You can operate the DE10-Nano in three ways:   

1) Serial Console
2) SSH
3) GUI

### Serial Console

1. Install a serial console application such as screen, [PuTTY](https://www.putty.org/ ), or minicom on your development PC. This example uses PuTTY.  

   ```
   sudo apt install -y putty
   ```

2. Before you power on the DE10-Nano, connect the following:

- Ethernet cable from DE10-Nano to a modem or router
- Power supply from DE10-Nano to an outlet
- USB cable (Mini-B to Type-A) from the DE10-Nano (UART-TO-USB port) to your development PC
  ![UART-TO-USB](picture/UART-TO-USB.png)
  

3. Using dmesg, identify the USB device path.

   Input:

   ```
   dmesg | tail
   ```

   Output:

   ```
   [12782.413742] usb 1-2.4: Detected FT232RL
   [12782.414335] usb 1-2.4: FTDI USB Serial Device converter now attached to ttyUSB0
   ```

   In this example, the path returned is **ttyUSB0**.


**Note**: If you cannot find any message, install FT2XXRL device driver [FTDI chip D2XX driver Installation](https://ftdichip.com/drivers/d2xx-drivers). 


4. Open PuTTY and set up a serial connection.  
   
   ```
   sudo putty
   ```
   
5. Set the speed to **115200** and use the USB device path for the serial line.
   
   ![PuTTY-serial-settings](picture/PuTTY-serial-settings.png)
   
6. Click **Open** and hit **Enter** to log in.  
   
   ![PuTTY-blackbox](picture/PuTTY-Blackbox.png)
   
   **Note**: Please reboot DE10-Nano if you are not able to connect.

   Default login credentials:  
   - Username: **root**  
   - Password: **de10nano**

### SSH

1. Before you power on the DE10-Nano, connect the following:

- Ethernet cable from DE10-Nano to a modem or router
- Power supply from DE10-Nano to an outlet
   
2. Boot DE10-Nano and wait about 30 seconds. If DHCP works, DE10-Nano will receive an IP address from your gateway during the boot process. You can use this IP address to log in    via ssh.

3. Use `ip addr` or `ifconfig` to confirm your network address.  
   
   Input:
   
   ```
   ip addr
   ```
   
   Output:
   
   ```
   ...
   2: eth0:
    link/ether 94:c6:91:a0:76:17 brd ff:ff:ff:ff:ff:ff
    inet 192.168.3.28/24 brd 192.168.3.255 scope global dynamic eno1
    ...
   ```
   
   In the above example, the host has 192.168.3.28/24 as the IP address. The DE10-Nano should have the same network address.  

4. Use the ping command to search for addresses used on the same network.  
   

   Input:

   ```
   echo 192.168.3.{1..254} | xargs -P255 -n1 ping -s1 -c1 -W1 | grep ttl
   ```

   Output:

   ```
   9 bytes from 192.168.3.1: icmp_seq=1 ttl=64
   9 bytes from 192.168.3.28: icmp_seq=1 ttl=64
   9 bytes from 192.168.3.98: icmp_seq=1 ttl=64
   ```
   
   In this example, the gateway router uses 192.168.3.1 and the DE10-Nano uses 192.168.3.98.


   **Note**: If you find several addresses on the same network, use `ip addr` to find the DE10-Nano network address.  


5. Log in to the console using:
   

   ```
   ssh root@192.168.3.98
   ```
   
   Default login credentials:  
   - Username: **root**  
   - Password: **de10nano** 

####  Set up a Static IP Address (Optional)

Because the DHCP server sets another IP address every time the DE10-Nano reboots, some users may want to set a static IP address. 

1. Use `ip addr` or `ifconfig` to confirm your network address.

2. To set up a new network connection, open a console on the DE10-Nano and type the following:

   Input:

    ```
    IPADDR=192.168.3.200/24
    GATEWAY=192.168.3.1
    DNS=8.8.8.8,8.8.4.4
    ```
3. Set the **IPADDR** and **GATEWAY** fields according to your network environment.

   In this example, **192.168.3.1** is used by the gateway router. Thus, 192.168.3.X/24 is the network address. The static IP, **192.168.3.200**, is provided to the DE10-Nano. **/24** is used after the IP address to identify a subnet mask.
    
3. Provide a connection name.

    Input:

    ```
    NAME=MyEthConnection
    ```

    This example uses "MyEthConnection".

4. Use `nmcli` to create a new network connection.

   Because Ubuntu 16.04 on the DE10-Nano uses NetworkManager, you can use `nmcli` or `nmtui`. 
    
   Input:

   ```
   nmcli connection add type ethernet autoconnect yes ifname eth0 con-name $NAME -- ipv4.method manual ipv4.address $IPADDR ipv4.dns $DNS ipv4.gateway $GATEWAY ipv6.method ignore connection.autoconnect-priority 1
   ```

   Output:

   ```
   Connection 'MyEthConnection' (d2bd3682-0b3d-4b27-88bc-6c52a4536d03) successfully added.
   ```

5. Reboot the DE10-Nano and check the IP address. If you see the IP address that you set, you can SSH with that IP address.

6. Connect to the DE10-Nano from a PC with the IP address.

   ```
   ssh root@192.168.3.200
   ```

   To delete the connection, use the following command:

   ```
   nmcli connection delete MyEthConnection
   ```
    
7. Set an SSH key to easily log in to DE10-Nano (Optional).
   

   Open a terminal on your development PC (host) and type your public key to DE10-Nano.

   ```
   ssh-copy-id root@192.168.3.200
   ```

   Set up the ssh config file.

   ```
   vim ~/.ssh/config
   ```

   Place it below.

   ```
   Host de10nano
     HostName 192.168.100.200
     User root
   ```
   
   This allows you to log in with the following keyword only.
   
   ```
   ssh de10nano
   ```

### GUI

To use the GUI, connect the following hardware to the DE10-Nano. See the image below for hardware setup. Make sure to connect the power supply from DE10-Nano to an outlet. 

<!-- does the GUI need an Ethernet cable -->

* HDMI cable and compatible display
* USB adapter (for type A to micro-B USB cable)
* USB hub
* USB keyboard
* USB mouse

![GUI Accessories](picture/GUI_Accessories.png)

![DE10-Nano-LXDE-image](picture/LXDE-Desktop.png)
 
## Step 3: Install Azure IoT Edge

For this step, the DE10-Nano needs to access the internet. Make sure the DE10-Nano is connected to a modem or router via an Ethernet cable.

To use Azure IoT Edge for managing IoT devices using container virtualization, you will need to install Azure IoT Edge on each device.

**Note**: See [Install the Azure IoT Edge runtime on Debian-based Linux systems][LINK_Azure_IoT_Edge] for Microsoft's tutorial on how to install Azure IoT Edge runtime on Linux devices if you need more information. 

### Set up apt Package Setting

1. Log in to the DE10-Nano and open a terminal.   

2. Open **/overlay/fpgaoverlay.sh**  with your favorite editor

   ```
   nano /overlay/fpgaoverlay.sh
   ```

3. Comment out all lines after mkdir  

   ```
   echo "creating $overlay_dir"
   mkdir $overlay_dir

   #echo "Doing Device Tree Overlay"
   #echo overlay.dtbo > $overlay_dir/path

   #echo "Successfully Device Tree Overlay Done"

   #echo "Loading altvipfb"
   #modprobe altvipfb
   #echo "Successfully altvipfb is loaded"
   ```

4. Save the change and exit the editor

5. Add package list
   
   ```
   mkdir -p ~/tmp && \
   cd ~/tmp && \
   curl https://packages.microsoft.com/config/ubuntu/18.04/multiarch/prod.list > ./microsoft-prod.list && \
   cp ./microsoft-prod.list /etc/apt/sources.list.d/ && \
   curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg && \
   cp ./microsoft.gpg /etc/apt/trusted.gpg.d/
   ```

6. Install Moby Engine, which Azure IoT Edge officially supports.
   
   ```
   apt update
   apt install -y moby-engine
   ```


### Install Azure IoT Edge Runtime

1. Install the Iot Edge Runtime
   
   ```
   curl -L https://github.com/Azure/azure-iotedge/releases/download/1.2.3/aziot-identity-service_1.2.2-1_ubuntu18.04_armhf.deb -o aziot-identity-service.deb && \
   curl -L https://github.com/Azure/azure-iotedge/releases/download/1.2.3/aziot-edge_1.2.3-1_ubuntu18.04_armhf.deb -o aziot-edge.deb && \
   sudo apt-get install -y ./aziot-identity-service.deb ./aziot-edge.deb
   ```
   

<!-- should we add a note about the current version? and need to update as time passes?) -->
    
<!-- replaced `apt install -y iotedge`) -->
  
## Step 4: Create an IoT Hub Instance on Azure Portal

You must register the DE10-Nano to the Azure cloud for IoT edge runtime to work.


1. Sign up for [Azure Portal][LINK_Azure_portal].

2. Log in to your Azure Portal.

3. Create a new resource group (recommended). From the Microsoft Azure homepage, navigate to **Azure services** > **Resource groups**, click **Add**.
   
   ![azure-grp](picture/azure-iot-00.png)
   
4. Set the project details (Subscription, Resource group name, Region). This example uses **de10-nano** for the resource group name.
   
   ![azure-grp-details](picture/azure-iot-01.png)

   **Note**: When you complete the tutorials in this series, you can delete the resource group to avoid unintentional costs. For cost details, see [Azure billing documentation][LINK_Azure_cost].

5. From the homepage, navigate to the search bar, type "iothub", and then select **IoT Hub**. 
   
   ![azure-iot-hub-search](picture/azure-iot-01-b.png)

6. Click **Add** to create an IoT Hub instance.
   
   ![azure-iot-hub-creation](picture/azure-iot-02.png)

7. Set the project details. For example, **Subscription** (Free Trial), **Resource group** (de10-nano), **Region** (US West 2), and **IoT Hub name** (de10-nano-iot-hub).
   
   ![azure-iot-hub-generate-instance](picture/azure-iot-03-a.png)

8. Click **Next: Networking** and choose a **Connectivity method**.

9. Click **Next: Management**, choose a **Pricing and scale tier**, and then click  **Review + create**.

   ![azure-iot-hub-choose-tier](picture/azure-iot-04.png)

   **Note**: The tier and number of units determines how many messages can be sent per day from the device to the Azure cloud. The F1: Free tier allows a maximum of 8000 messages per day.

10. Review the details of the IoT Hub and then click **Create**. The deployment of your IoT Hub may take a few minutes.
  
      ![azure-iot-hub-instantiate](picture/azure-iot-05.png)

11. When you see a message that **Your deployment is complete**, click **Go to resource**.
 
      ![azure-iot-hub-go-to-resource](picture/azure-iot-08.png)

### Register DE10-Nano as an IoT Edge Device

In this step, you will register the DE10-Nano as an IoT Edge device. 

1. Navigate to **Automatic Device Management** or **IoT Edge** and click on **IoT Edge** as shown below.

   ![azure-iot-hub-register-iot-edge](picture/azure-iot-10.png)

2. Click **Add an IoT Edge device**.

   ![azure-iot-hub-add-iot-edge-device](picture/azure-iot-11.png)

3. Under **Device ID**, set the name of your device (For example, **de10-nano-device**). Click **Save**. 
   
   ![azure-iot-hub-create-a-device](picture/azure-iot-12.png)

4. Select your device.
   
   ![azure-iot-hub-see-device-information](picture/azure-iot-13.png)
   
5. Copy the Primary Connection String. You can also choose to copy the Secondary Connection String. With the connection string, Azure cloud can confirm that DE10-Nano is a         valid device.  
   
   Below is an example of the connection string, which varies from device to device.
   
   ```
   HostName=de10nano-iothub.azure-devices.net;DeviceId=de10-nano-iotedge;SharedAccessKey=rPiy9a15CM4WQ54EAwXq6/XQ07diE0zUi0NXTCBmuic=
   ```
   
   ![azure-iot-hub-copy-connection-string](picture/azure-iot-14.png)

## Step 5: Update the IoT Edge Daemon Config File

Before running IoT Edge runtime, the connection string must be copied into the IoT Edge configuration file on DE10-Nano. If IoT Edge runtime on DE10-Nano is executed without setting the connection string, the application will not start.


1. Open a terminal on the DE10-Nano using SSH, a serial console, or the GUI. 
2. Copy and open IoT Edge Runtime configuration file
   ```
   cp /etc/aziot/config.toml.edge.template /etc/aziot/config.toml && \
   nano /etc/aziot/config.toml 
   ```
   
3.  Edit the config.toml by uncommenting the line below and insert the connection string in **Register DE10-Nano as an IoT Edge Device** Step5.
      ```
      # [provisioning]
      # source = "manual"
      # connection_string = "HostName=example.azure-devices.net;DeviceId=my-device;SharedAccessKey=YXppb3QtaWRlbnRpdHktc2VydmljZXxhemlvdC1pZGU="

      ```
4. Update IoT Edge runtime with:
      ```
      sudo iotedge config apply
      ```

      ```
      root@de10nano:/etc/aziot# sudo iotedge config apply
      Note: Symmetric key will be written to /var/secrets/aziot/keyd/device-id
      Azure IoT Edge has been configured successfully!

      Restarting service for configuration to take effect...
      Stopping aziot-edged.service...Stopped!
      Stopping aziot-identityd.service...Stopped!
      Stopping aziot-keyd.service...Stopped!
      Stopping aziot-certd.service...Stopped!
      Stopping aziot-tpmd.service...Stopped!
      Starting aziot-edged.mgmt.socket...Started!
      Starting aziot-edged.workload.socket...Started!
      Starting aziot-identityd.socket...Started!
      Starting aziot-keyd.socket...Started!
      Starting aziot-certd.socket...Started!
      Starting aziot-tpmd.socket...Started!
      Starting aziot-edged.service...Started!
      Done.
      ```
5. Verify successful installation
   ```
      root@de10nano:~# iotedge list
      NAME             STATUS           DESCRIPTION      CONFIG
      edgeAgent        running          Up 6 minutes     mcr.microsoft.com/azureiotedge-agent:1.2
   ```
      

## Step 6: Set up DNS and Log Policy

### Set up DNS on Container Runtime

1. Check the IoT Edge runtime health.
    
   Input:

   ```
   iotedge check
   ```

   Output:

   ```
   Configuration checks (aziot-identity-service)
   ---------------------------------------------
   √ keyd configuration is well-formed - OK
   √ certd configuration is well-formed - OK
   √ tpmd configuration is well-formed - OK
   √ identityd configuration is well-formed - OK
   √ daemon configurations up-to-date with config.toml - OK
   √ identityd config toml file specifies a valid hostname - OK
   √ aziot-identity-service package is up-to-date - OK
   √ host time is close to reference time - OK
   √ preloaded certificates are valid - OK
   √ keyd is running - OK
   √ certd is running - OK
   √ identityd is running - OK
   √ read all preloaded certificates from the Certificates Service - OK
   √ read all preloaded key pairs from the Keys Service - OK
   √ ensure all preloaded certificates match preloaded private keys with the same ID - OK

   Connectivity checks (aziot-identity-service)
   --------------------------------------------
   √ host can connect to and perform TLS handshake with iothub AMQP port - OK
   √ host can connect to and perform TLS handshake with iothub HTTPS / WebSockets port - OK
   √ host can connect to and perform TLS handshake with iothub MQTT port - OK

   Configuration checks
   --------------------
   √ aziot-edged configuration is well-formed - OK
   √ configuration up-to-date with config.toml - OK
   √ container engine is installed and functional - OK
   √ configuration has correct URIs for daemon mgmt endpoint - OK
   √ aziot-edge package is up-to-date - OK
   √ container time is close to host time - OK
   ‼ DNS server - Warning
      Container engine is not configured with DNS server setting, which may impact connectivity to IoT Hub.
      Please see https://aka.ms/iotedge-prod-checklist-dns for best practices.
      You can ignore this warning if you are setting DNS server per module in the Edge deployment.
   √ production readiness: container engine - OK
   ‼ production readiness: logs policy - Warning
      Container engine is not configured to rotate module logs which may cause it run out of disk space.
      Please see https://aka.ms/iotedge-prod-checklist-logs for best practices.
      You can ignore this warning if you are setting log policy per module in the Edge deployment.
   ‼ production readiness: Edge Agent's storage directory is persisted on the host filesystem - Warning
      The edgeAgent module is not configured to persist its /tmp/edgeAgent directory on the host filesystem.
      Data might be lost if the module is deleted or updated.
      Please see https://aka.ms/iotedge-storage-host for best practices.
   × production readiness: Edge Hub's storage directory is persisted on the host filesystem - Error
      Could not check current state of edgeHub container
   √ Agent image is valid and can be pulled from upstream - OK

   Connectivity checks
   -------------------
   √ container on the default network can connect to upstream  AMQP port - OK
   √ container on the default network can connect to upstream HTTPS / WebSockets port - OK
   √ container on the default network can connect to upstream MQTT port - OK
   √ container on the IoT Edge module network can connect to upstream AMQP port - OK
   √ container on the IoT Edge module network can connect to upstream HTTPS / WebSockets port - OK
   √ container on the IoT Edge module network can connect to upstream MQTT port - OK
   32 check(s) succeeded.
   3 check(s) raised warnings. Re-run with --verbose for more details.
   1 check(s) raised errors. Re-run with --verbose for more details.


    ```
    
    In this example, there are a few warnings and one error for IoT Edge.
     
    - If the DHCP server does not assign a DNS address, you will need to assign a DNS address. If you do not receive a DNS warning, no action is needed.
    - In this tutorial, you can ignore the following:
      - Certificate warnings  
      - Logs policy warnings. As needed, you can set these.
      - Edge Hub error. 
      
    
    **Note**: The Edge Hub error occurs when there has never been a deployment on an IoT Edge device. If you deploy a container from the Azure cloud, the Edge Hub container is automatically generated and this error will go away. This is a known issue, see [Microsoft Github Issue No.36941](https://github.com/MicrosoftDocs/azure-docs/issues/36941).

2. Make a new folder.

    ```
    mkdir /etc/docker
    ```

3. Open `daemon.json`.

    ```
    vim /etc/docker/daemon.json
    ```

4. Copy and paste the code below into the daemon.json file. Use `:wq!` to save your changes.

    ```
    {
        "dns": ["8.8.8.8", "8.8.4.4"]
    }
    ```
    
    `8.8.8.8` and `8.8.4.4` are public DNS addresses provided by Google. In this tutorial, they are used as DNS servers.

### Set up Log Policy
   

1. Put log settings after DNS setting.
  
    ```
    vim /etc/docker/daemon.json
    ```
    
    ```
    {
        "dns": ["8.8.8.8", "8.8.4.4"],
        "log-driver": "json-file",
        "log-opts": {
        "max-size": "10m",
        "max-file": "3"
        }
    }
    ```
    **Note**: Be sure to copy the above syntax exactly since incorrect syntax will disconnect the device.

2. Restart the container runtime.

    ```
    systemctl restart docker
    ```

    In this step, container runtime and Azure IoT Edge are installed. Also, DE10-Nano is registered on the Azure cloud, allowing you to deploy an application from the Azure cloud to the DE10-Nano.

3. Check Azure IoT Edge runtime again.

    Input:

    ```
    iotedge check
    ```

    Output:

    ```
      ...
      34 check(s) succeeded.
      1 check(s) raised warnings. Re-run with --verbose for more details.
      1 check(s) raised errors. Re-run with --verbose for more details.

    ```

   The DNS warning and logs policy warning have disappeared. You can continue to ignore the Edge Hub error.
   
## Step 7: Expand microSD Card Storage Size

Containers often consume microSD card storage, we recommend that you to expand your microSD card partition. The scripts in this step expand your card partition to the maximum storage capacity of the microSD card.


1. Execute the following shell script on DE10-Nano to expand the storage size.

    ```
    ~/expand_rootfs.sh
    ```

2. Reboot the DE10-Nano

    ```
    reboot
    ```

3. Execute the following shell script after you reboot.

    ```
    ~/resize2fs_once.sh
    ```

4. Check the partition size.

    Input:

    ```
    df -Th
    ```

    Output:
    
    ```
    Filesystem     Type      Size  Used Avail Use% Mounted on
    /dev/root      ext3       15G  2.0G   12G  15% /
    ```

    In this example, the storage is extended to 12GB for a 16GB microSD card. 

## Step 8: Deploy a Simulated Temperature Sensor

In this step, you deploy your first module from the Azure Marketplace. The module that you deploy in this step simulates a temperature sensor and sends simulated data to the cloud. 

**Note**: See [Quickstart: Deploy your first IoT Edge module to a virtual Linux device][LINK_Azure_deploy_a_module] for Microsoft's tutorial on how to remotely deploy a module to an IoT Edge device. This step generally follows the instructions in the Microsoft tutorial. 
  

1. Open [Azure Portal][LINK_Azure_portal] and search for "Simulated". Under **Marketplace**, click on **Simulated Temperature Sensor**.

    ![azure-iot-hub-find-temperature-module](picture/azure-tmp-00.png)

2. Set the Subscription, IoT Hub, and IoT Edge Device Name. Click on **Find Device** and devices which belong to your specified IoT Hub will appear.  

3. Click **Create**.  

    ![azure-iot-hub-specify-iot-edge](picture/azure-tmp-01.png)

4. Click on **Runtime Settings** and make sure the  **Edge Agent** Image URI is set to *mcr.microsoft.com/azureiotedge-agent:1.2*.
![azure-iot-hub-set-runtime-settings](picture/azure-tmp-06.png)


5. Make sure that **SimulatedTemperatureSensor** exists in **IoT Edge Modules** and then click on **Next: Routes**.

    ![azure-iot-hub-set-module](picture/azure-tmp-02.png)
    
6. Keep the defaults set for routes and then click **Review + create**.
    
    ![azure-iot-hub-routes](picture/azure-tmp-03.png)

7. Click **Create**. The sensor module will be sent to your device and deployed automatically.  

    ![azure-iot-hub-set-routes](picture/azure-tmp-04.png)

8. Navigate to the device page. You will see the deployed sensor module.

    ![azure-iot-hub-iot-edge](picture/azure-tmp-05.png)

9. Open a DE10-Nano console and use `iotedge list` to confirm that the module is deployed (that is, STATUS: running).

    Input:

    ```
    iotedge list
    ```

    Output:

    ```
    NAME                        STATUS           DESCRIPTION      CONFIG
    SimulatedTemperatureSensor  running          Up 2 minutes     mcr.microsoft.com/azureiotedge-simulated-temperature-sensor:1.0
   edgeAgent                   running          Up 2 minutes     mcr.microsoft.com/azureiotedge-agent:1.2
   edgeHub                     running          Up 2 minutes     mcr.microsoft.com/azureiotedge-hub:1.1
    ```

    "SimulatedTemperatureSensor" should display, indicating that you deployed the container module on your DE10-Nano.  



The first deployment may take some time. Use the `iotedge list` command several times and wait until **edgeHub**, **edgeAgent** and **SimulatedTemperatureSensor** are listed as above. edgeHub and edgeAgent are containers for managing your IoT Edge.


### Check Container Data from the DE10-Nano Device

1. Check the IoT Edge runtime log to make sure that the module is sending simulated data to the Azure cloud.  

    
    Input:

    ```
    iotedge logs SimulatedTemperatureSensor | tail -n 3
    ```
    
    Output:

    ```
    02/14/2020 07:10:29> Sending message: 499, Body:
    [{"machine":{"temperature":109.61642031237317,"pressure":11.095541554574158},"ambient":{"temperature":20.776358844375405,"humidity":26},"timeCreated":"2020-02-14T07:10:29.6590332Z"}]
    02/14/2020 07:10:34> Sending message: 500, Body: [{"machine":{"temperature":109.47054543368078,"pressure":11.078922897507937},"ambient":{"temperature":20.883418869871374,"humidity":25},"timeCreated":"2020-02-14T07:10:34.6897821Z"}]
    Done sending 500 messages
    ```
    
    The module will stop after sending 500 messages and you will need to restart it.  
    
    ```
    iotedge restart SimulatedTemperatureSensor
    ```

### Check Container Data from the Cloud

Confirm that the sensor module runs on the DE10-Nano by checking it from the Azure cloud. 

#### Install Azure CLI

This step uses a development PC with Ubuntu 16.04 and follows the Azure CLI installation guide for Ubuntu.  

**Note**: See the [Azure Command-Line Interface (CLI) documentation][LINK_Azure_CLI] for Microsoft's tutorial on how to install the Azure CLI. 


1. From your development PC, download and execute the install script.

    ```
    curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
    ```

    This script automatically installs the Azure CLI application on your PC.

2. Enter your authentication information in a browser window.

    ```
    az login
    ```

**Note**: If you have an "xdg-open" error, make sure that BROWSER environment variable is set.  

    ```
    echo $BROWSER

    export BROWSER=firefox
    ```

3. Install the Azure IoT extension for Azure CLI.
   
   Microsoft provides an open-source extension, [Azure IoT extension for Azure CLI](https://github.com/Azure/azure-iot-cli-extension), for developing Azure IoT applications with Azure CLI.

    In this tutorial, we use it to see messages that the IoT Hub has received.
    
    ```
    az extension add --name azure-iot
    ```

**Note**: If you re-open your terminal, the completion feature will be effective automatically. 

#### Confirm DE10-Nano Messages with Azure CLI

1. Monitor DE10-Nano messages with Azure CLI

    Since you have linked the Azure CLI to your cloud account, you can use Azure CLI to monitor data received by the IoT Hub cloud. Put the name of your IoT Hub after `--hub-name` in the following command. For example, de10-nano-iot-hub.
    
    Input:

    ```
    az iot hub monitor-events --hub-name de10-nano-iot-hub
    ```
    
    Output:

    ```
    ...
    {
        
        "event": {
            "origin": "de10-nano-iotedge",
            "payload": "{\"machine\":{\"temperature\":104.63849517546522,\"pressure\":10.528436159230216},\"ambient\":{\"temperature\":20.712849819666172,\"humidity\":25},\"timeCreated\":\"2020-02-14T06:48:35.4649348Z\"}"
        }
    }
    {
        "event": {
            "origin": "de10-nano-iotedge",
            "payload": "{\"machine\":{\"temperature\":104.8690607204889,\"pressure\":10.554703120055697},\"ambient\":{\"temperature\":21.158996682920957,\"humidity\":26},\"timeCreated\":\"2020-02-14T06:48:40.5055251Z\"}"
        }
    }
    ...
    ```

**Note**: If you cannot see any messages, use `iotedge logs` to check if the SimulatedTemperatureSensor module is still working.
    
    
## Next Steps

Congratulations! You completed the first tutorial in this series. To continue to the next tutorial, go to [Create a Container Using Visual Studio Code and Deploy it to the DE10-Nano][LINK_module_02]. 

If you plan to continue to the next tutorial in this series, do not delete the resource group you created earlier. If not, you may now delete it. When you delete the resource group, you also delete all Azure services you associated with it.


[LINK_Azure_cost]: https://docs.microsoft.com/en-us/azure/billing/billing-understand-your-bill
[LINK_Terasic_DE10_NANO_User_Manual]: https://www.terasic.com.tw/cgi-bin/page/archive_download.pl?Language=English&No=1046&FID=f1f656bb5f040121c36f2f93f6b107ff
[LINK_Terasic_bootable]: http://download.terasic.com/downloads/cd-rom/de10-nano/DE10-Nano-Cloud-Native.zip
[LINK_Terasic_DE10-Nano-Purchase]: https://de10-nano.terasic.com/
[LINK_Terasic_getting_started_guide]: https://www.terasic.com.tw/attachment/archive/1046/Getting_Started_Guide.pdf
[LINK_Azure_IoT_Edge]: https://docs.microsoft.com/en-us/azure/iot-edge/how-to-install-iot-edge-linux
[LINK_Azure_portal]: https://portal.azure.com/
[LINK_Azure_CLI]: https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest
[LINK_Azure_IoT_Edge_Troubleshoot]: https://docs.microsoft.com/en-us/azure/iot-edge/troubleshoot
[LINK_Azure_deploy_a_module]: https://docs.microsoft.com/en-us/azure/iot-edge/quickstart-linux#deploy-a-module

[LINK_Macnica_DE10_Link]: https://www.macnica.co.jp/business/semiconductor/articles/intel/2075/

<!---

[LINK_module_02]: https://TODO.link.to.module2

--->
