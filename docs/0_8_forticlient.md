## Connecting from FortiClient VPN client

For FortiGate administrators, a free version of FortiClient VPN is available which supports basic IPsec and SSL VPN and does not require registration with EMS. This version does not include central management, technical support, or some advanced features.

### Downloading and installing the standalone FortiCient VPN client

You can download the free VPN client from [FNDN](https://fndn.fortinet.net/index.php?/category/1-fortianswers/) or [FortiClient.com](https://www.forticlient.com/).

When the free VPN client is run for the first time, it displays a disclaimer. You cannot configure or create a VPN connection until you accept the disclaimer and click I accept:

![](https://fortinetweb.s3.amazonaws.com/docs.fortinet.com/v2/resources/5ec8a15f-aa17-11ec-9fd1-fa163e15d75b/images/39577d294e22974ebffab69f647b2ba9_first_run.png)

### Configuring an SSL VPN connection
To configure an SSL VPN connection:

1. On the Remote Access tab, click on the settings icon and then Add a New Connection.

![](https://fortinetweb.s3.amazonaws.com/docs.fortinet.com/v2/resources/5ec8a15f-aa17-11ec-9fd1-fa163e15d75b/etc/a8664acf6ade441f9cfd804f16a04ec6_SSL%20VPN%20NEW.PNG)

2. Select SSL-VPN, then configure the following settings:

> Connection Name     SSLVPNtoHQ
> Description         (Optional)
> Remote Gateway      172.20.120.123
> Customize port      10443
> Client Certificate  Select Prompt on connect or the certificate from the dropdown list.
> Authentication      Select Prompt on login for a prompt on the connection screen

3. Click Save to save the VPN connection.

### Connecting to SSL VPN
To connect to SSL VPN:

1. On the Remote Access tab, select the VPN connection from the dropdown list. Optionally, you can right-click the FortiTray icon in the system tray and select a VPN configuration to connect.

2. Enter your username and password.
3. Click the Connect button.
4. After connecting, you can now browse your remote network. Traffic to 192.168.1.0 goes through the tunnel while other traffic goes through the local gateway. FortiClient displays the connection status, duration, and other relevant information.
5. Click the Disconnect button when you are ready to terminate the VPN session.


### Checking the SSL VPN connection
To check the SSL VPN connection using the GUI:

1. On the FortiGate, go to VPN > Monitor > SSL-VPN Monitor to verify the list of SSL users.
2. On the FortiGate, go to Log & Report > Forward Traffic to view the details of the SSL entry.

To check the tunnel log in using the CLI:

> get vpn ssl monitor
> SSL VPN Login Users:
>  Index   User          Auth Type    Timeout   From           HTTP in/out  HTTPS in/out
>  0       sslvpnuser1   1(1)         291       10.1.100.254   0/0          0/0
> 
> SSL VPN sessions:
>  Index   User          Source IP      Duration   I/O Bytes      Tunnel/Dest IP 
>  0       sslvpnuser1   10.1.100.254   9          22099/43228    10.212.134.200