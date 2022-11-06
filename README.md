# Linux Bluetooth with Plugin.BlueZ

> ## Announcement
>
> The project is officially changing names to, [Linux.Bluetooth](https://github.com/SuessLabs/Linux.Bluetooth)!
>
> This repository will remain and eventually marked as, _read-only_. Please migrate your project over to the new [NuGet package](https://www.nuget.org/packages/Linux.Bluetooth) to continue getting the latest features and builds.
>
> A .NET BluetoothLE library for Linux using BlueZ's D-Bus APIs.

[![Linux.Bluetooth NuGet Badge](https://buildstats.info/nuget/Linux.Bluetooth?dWidth=70&includePreReleases=true)](https://www.nuget.org/packages/Linux.Bluetooth/)

[![Plugin.BlueZ NuGet Badge](https://buildstats.info/nuget/Plugin.BlueZ?dWidth=70&includePreReleases=true)](https://www.nuget.org/packages/Plugin.BlueZ/)

The library uses, [Tmds.DBus](https://github.com/tmds/Tmds.DBus) to access Linux's D-Bus, the preferred interface for Bluetooth in userspace.

Check out the SuessLabs article on using [Plugin.BlueZ](https://suesslabs.com/csharp/net-and-linux-bluetooth/)

## Requirements

* Linux
* A recent release of BlueZ. This package was tested with BlueZ 5.50. You can check which version you're using with `bluetoothd -v`.

### Supported Distributions

Plugin.BlueZ aims to support Linux Distributions where both .NET and BlueZ is supported. Officially, this NuGet package has only been tested on Ubuntu 20.04.

List of [BlueZ supported](http://www.bluez.org/about/) distros:

* Ubuntu Linux
* Raspbian (_Raspberry PI_)
* Debian GNU/Linux
* Fedora Core / Red Hat Linux
* OpenSuSE / SuSE Linux
* Mandrake Linux
* Gentoo Linux
* Chrome OS

## Installation

```bash
dotnet add package Plugin.BlueZ
```

## Usage

C# events are available for several properties. Events are useful for properly handling disconnects and reconnects.

### Get a Bluetooth adapter

```C#
using Plugin.BlueZ;
...
IAdapter1 adapter = (await BlueZManager.GetAdaptersAsync()).FirstOrDefault();
```

or get a particular adapter:

```C#
IAdapter1 adapter = await BlueZManager.GetAdapterAsync(adapterName: "hci0");
```

## Scan for Bluetooth devices

```C#
adapter.DeviceFound += adapter_DeviceFoundAsync;

await adapter.StartDiscoveryAsync();
...
await adapter.StopDiscoveryAsync();
```

### Get Devices

`adapter.DeviceFound` (above) will be called immediately for existing devices, and as new devices show up during scanning; `eventArgs.IsStateChange` can be used to distinguish between existing and new devices. Alternatively you can can use `GetDevicesAsync`:

```C#
IReadOnlyList<Device> devices = await adapter.GetDevicesAsync();
```

### Connect to a Device

```C#
device.Connected += device_ConnectedAsync;
device.Disconnected += device_DisconnectedAsync;
device.ServicesResolved += device_ServicesResolvedAsync;

await device.ConnectAsync();
```

Alternatively, you can wait for "Connected" and "ServicesResolved" to equal true:

```C#
TimeSpan timeout = TimeSpan.FromSeconds(15);

await device.ConnectAsync();
await device.WaitForPropertyValueAsync("Connected", value: true, timeout);
await device.WaitForPropertyValueAsync("ServicesResolved", value: true, timeout);
```

### Retrieve a GATT Service and Characteristic

Prerequisite: You must be connected to a device and services must be resolved. You may need to pair with the device in order to use some services.

Example using GATT Device Information Service UUIDs.

```C#
string serviceUUID = "0000180a-0000-1000-8000-00805f9b34fb";
string characteristicUUID = "00002a24-0000-1000-8000-00805f9b34fb";

IGattService1 service = await device.GetServiceAsync(serviceUUID);
IGattCharacteristic1 characteristic = await service.GetCharacteristicAsync(characteristicUUID);
```

### Read a GATT Characteristic value

```C#
byte[] value = await characteristic.ReadValueAsync(timeout);

string modelName = Encoding.UTF8.GetString(value);
```

### Subscribe to GATT Characteristic Notifications

```C#
characteristic.Value += characteristic_Value;
...

private static async Task characteristic_Value(GattCharacteristic characteristic, GattCharacteristicValueEventArgs e)
{
  try
  {
    Console.WriteLine($"Characteristic value (hex): {BitConverter.ToString(e.Value)}");

    Console.WriteLine($"Characteristic value (UTF-8): \"{Encoding.UTF8.GetString(e.Value)}\"");
  }
  catch (Exception ex)
  {
    Console.Error.WriteLine(ex);
  }
}
```

## Tips

It may be necessary to pair with a device for a GATT service to be visible or for reading GATT characteristics to work. To pair, one option is to run `bluetoothctl` (or `sudo bluetoothctl`)
and then run `default agent` and `agent on` within `bluetoothctl`. Watch `bluetoothctl` for pairing requests.

See [Ubuntu's Introduction to Pairing](https://core.docs.ubuntu.com/en/stacks/bluetooth/bluez/docs/reference/pairing/introduction).

### BluetoothCtl Helper

From command line, use `bluetoothctl` or Bluetooth Manager to scan and retrieve device UUIDs and Services to assist with debugging.."

```bash
$ bluetoothctl

; Scan for devices
scan on

; Stop Scanning
scan off

; List known devices
devices
```

## Generating

Tmds.DBus.Tool is used to generate the D-Bus object interfaces.

## Contributing

See [Contributing](./github/CONTRIBUTING.md).

## Reference

* [Doing Bluetooth Low Energy on Linux](https://elinux.org/images/3/32/Doing_Bluetooth_Low_Energy_on_Linux.pdf)
* **BlueZ API**:
  * [HEAD](https://git.kernel.org/pub/scm/bluetooth/bluez.git/tree/doc)
  * [v5.53](https://git.kernel.org/pub/scm/bluetooth/bluez.git/tree/doc?h=5.53) - _i.e. Ubuntu v20.04 LTS_
* [BlueZ Official Site](http://www.bluez.org/)
* [Install BlueZ on the Raspberry PI](https://learn.adafruit.com/install-bluez-on-the-raspberry-pi/overview)
