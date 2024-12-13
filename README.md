# Hestia ASPNET/NET CORE Templates
The goal of this repository is to bring you a base template that allows you to host/run ASPNET/Net Core Applications with [Hestia Control Panel](https://hestiacp.com/).

## Requisites
Before you use theses templates make sure you have installed the runtime packages or SDK on your system. You can follow these links for more info.

- [Install .NET on Linux](https://learn.microsoft.com/en-us/dotnet/core/install/linux)
- [Host ASP.NET Core on Linux with Nginx](https://learn.microsoft.com/en-us/aspnet/core/host-and-deploy/linux-nginx)

An alternative to creating a service definition file for `systemd` if you don't have root access is to add the following to your crontab file: 

```bash
@reboot cd ~/netcoreapp; ./BlazorApp1 > log.txt`
```

You will need to change both the working directory and executable name correspondingly.

## Installation
The process is very simple. Only copy/clone the follow files:
* `netcore5000.sh` shell script that runs the app
* `netcore5000.tpl` http nginx proxy template
* `netcore5000.stpl` https nginx proxy template
* `netcoresock.sh` shell script that runs the app on Unix Sockets
* `netcoresock.tpl` shell script that runs the app on Unix Sockets
* `netcoresock.stpl` shell script that runs the app on Unix Sockets

To the the Hestia Nginx's templates, usually `/usr/local/hestia/data/templates/web/nginx`. Make sure the bash scripts have the proper run permissions. You can set them with the following command:
```bash
sudo chmod 755 /usr/local/hestia/data/templates/nginx/netcore*.sh
```

Once you have done the previous steps go to your control and select the web site. Once you selected it click on **Advanced options** and change the **Proxy template** to **netcore5000** or **netcoresock**. Then you can place your NetCore/ASPNET app on the **netcoreapp** folder, using the file manager.

## Notes
Keep in mind that the proxy template `netcore5000.sh` runs the application as usually NetCore app does. That means the TCP port 5000 will be locked for other applications and you can only run only one app on this port. I suggest you use Unix Sockets instead. Or create different templates for different TPC ports.

If you choose Unix Sockets to run you application you should add the following code to your app on `Program.cs`:

```c#
const string unixSocketPath = "/home/hestiaUser/web/domain/netcoreapp/netcore.sock";

builder.WebHost.ConfigureKestrel(options =>
{
    options.ListenUnixSocket(unixSocketPath);
});
```

You must replace *hestiaUser* and *domain* for the proper values. Or you can pass the proper arguments to build the right path, for example:

```c#
string unixSocketPath = $"/home/{args[1]}/web/{args[2]}/netcoreapp/netcore.sock";
```

Also you must keep in mind that when using UNIX sockets on Linux, the socket isn't automatically deleted on app shutdown. You must take control of this. The solution could be to check if file exists before build the `WebHost` :

```c#
if (File.Exists(UnixSocketPath))
{
    File.Delete(UnixSocketPath);
}

var builder = WebApplication.CreateBuilder(args);
builder.WebHost.ConfigureKestrel(options =>
{
    options.ListenUnixSocket(UnixSocketPath);
});
```
