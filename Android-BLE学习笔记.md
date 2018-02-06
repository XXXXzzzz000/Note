# 蓝牙BLE学习

学习资源地址:<https://developer.android.com/guide/topics/connectivity/bluetooth-le.html>

## 关键术语和概念

以下是关键的BLE术语和概念的总结：

- **通用属性配置文件(Generic Attribute Profile (GATT))** - GATT配置文件是通过BLE连接发送和接收被称为“属性”的短片段的通用规范。目前所有的低能耗应用程序都基于GATT。
  - 蓝牙SIG 为低能耗设备定义了许多[配置文件](https://www.bluetooth.com/specifications)。配置文件是设备如何在特定应用程序中工作的规范。请注意，设备可以实现多个配置文件。例如，一个设备可以包含一个心率监测器和一个电池电量检测器。
- **属性协议(Attribute Protocol (ATT))** -GATT建立在属性协议（ATT）之上。这也被称为GATT / ATT。ATT经过优化，可在BLE设备上运行。为此，它使用尽可能少的字节。每个属性由一个通用唯一标识符（UUID）唯一标识，该标识符是用于唯一标识信息的字符串ID的标准化128位格式。ATT传输的属性被格式化为特征和服务。
- **特性(Characteristic)** - 一个特性包含描述特性值的单个值和0-n个描述符。一个特征可以被认为是一个类，类似于一个阶级。 
- **描述符(Descriptor)** - 描述符是描述特征值的定义属性。例如，一个描述符可以指定一个人可读的描述，一个特征值的可接受范围，或者一个特征值特有的度量单位。
- **服务(Service)** - 服务是一个特征的集合。例如，您可以拥有一个名为“心率监测器”的服务，其中包含“心率测量”等特性。您可以在[bluetooth.org](https://www.bluetooth.com/specifications)上找到现有基于GATT的配置文件和服务的列表 。

## 角色和责任

以下是Android设备与BLE设备交互时适用的角色和职责：

- **中央与周边(Central vs. peripheral)**。这适用于BLE连接本身。处于中心角色的设备扫描，寻找广告，并且在外围角色中的设备进行广告。
- **GATT服务器与GATT客户端(server vs. client)**。这决定了两台设备在建立连接后如何相互通话。

为了理解这个区别，假设你有一个Android手机和一个BLE设备的活动追踪器。手机支持中心角色; 活动跟踪器支持外设角色（建立一个BLE连接，对等设备-两个central或两个peripheral角色之间是不能建立BLE连接的）。

一旦手机和活动追踪器建立了连接，他们就开始将GATT元数据转移到另一个。根据他们传输的数据的种类，其中一个或另一个可能充当服务器。例如，如果活动跟踪器想要将传感器数据报告给手机，则活动跟踪器可以充当服务器。如果活动跟踪器想要从手机接收更新，那么手机将作为服务器。

在本文档中使用的示例中，Android应用程序（在Android设备上运行）是GATT客户端。该应用程序从GATT服务器获取数据，GATT服务器是支持心率档案的BLE心率监测器 。但你也可以设计你的Android应用程序来扮演GATT服务器的角色。查看[BluetoothGattServer](https://developer.android.com/reference/android/bluetooth/BluetoothGattServer.html)更多信息。

## BLE权限

为了在您的应用程序中使用蓝牙功能，您必须声明蓝牙许可[BLUETOOTH](https://developer.android.com/reference/android/Manifest.permission.html#BLUETOOTH)。您需要此权限才能执行任何蓝牙通信，例如请求连接，接受连接以及传输数据。

如果您希望您的应用启动设备发现或操纵蓝牙设置，则还必须声明该[BLUETOOTH_ADMIN](https://developer.android.com/reference/android/Manifest.permission.html#BLUETOOTH_ADMIN) 权限。**注意：**如果您使用[BLUETOOTH_ADMIN](https://developer.android.com/reference/android/Manifest.permission.html#BLUETOOTH_ADMIN)权限，那么您还必须拥有[BLUETOOTH](https://developer.android.com/reference/android/Manifest.permission.html#BLUETOOTH)权限。

在应用程序清单文件中声明蓝牙许可。例如：

```xml
<uses-permission android：name = “android.permission.BLUETOOTH” /> <uses-permission android：name = “android.permission.BLUETOOTH_ADMIN” /> 
```
如果您想声明您的应用仅适用于具有BLE功能的设备，请在应用的清单中包含以下内容：

```xml
<uses-feature android：name = “android.hardware.bluetooth_le” android：required = “true” />  
```

但是，如果您希望将应用程序提供给不支持BLE的设备，则应该将此元素包含在应用程序的清单中，但设置`required="false"`。然后在运行时，您可以使用[PackageManager.hasSystemFeature()](https://developer.android.com/reference/android/content/pm/PackageManager.html#hasSystemFeature(java.lang.String))以下方法确定BLE可用性 ：

```java
// 使用此检查来确定设备上是否支持BLE
// 然后您可以选择性地禁用BLE相关功能。
if (!getPackageManager().hasSystemFeature(PackageManager.FEATURE_BLUETOOTH_LE)) {
    Toast.makeText(this, R.string.ble_not_supported, Toast.LENGTH_SHORT).show();
    finish();
}
```

**注意：** LE信标往往与位置有关。为了在 [BluetoothLeScanner](https://developer.android.com/reference/android/bluetooth/le/BluetoothLeScanner.html)没有过滤器的情况下使用，您必须通过在应用的清单文件中声明[ACCESS_COARSE_LOCATION](https://developer.android.com/reference/android/Manifest.permission.html#ACCESS_COARSE_LOCATION)或 [ACCESS_FINE_LOCATION](https://developer.android.com/reference/android/Manifest.permission.html#ACCESS_FINE_LOCATION)权限来请求用户的许可 。没有这些权限，扫描将不会返回任何结果。

## 设置BLE

在您的应用程序可以通过BLE进行通信之前，您需要验证设备是否支持BLE，如果是，请确保已启用BLE。请注意，只有在`<uses-feature.../>` 设置为`false` 时才需要进行此项检查。

如果不支持BLE，那么您应该优雅地禁用任何BLE功能。如果BLE支持但禁用，则可以请求用户启用蓝牙，而不必离开您的应用程序。这个设置分两步完成，使用[BluetoothAdapter](https://developer.android.com/reference/android/bluetooth/BluetoothAdapter.html)。

1. 得到 [BluetoothAdapter](https://developer.android.com/reference/android/bluetooth/BluetoothAdapter.html)

   这[BluetoothAdapter](https://developer.android.com/reference/android/bluetooth/BluetoothAdapter.html)是任何和所有的蓝牙活动所必需的。该[BluetoothAdapter](https://developer.android.com/reference/android/bluetooth/BluetoothAdapter.html)代表设备自己的蓝牙适配器（蓝牙无线电）。整个系统有一个蓝牙适配器，您的应用程序可以使用这个对象与它进行交互。下面的代码展示了如何获取适配器。请注意，此方法用于[getSystemService()](https://developer.android.com/reference/android/content/Context.html#getSystemService(java.lang.Class<T>))返回实例[BluetoothManager](https://developer.android.com/reference/android/bluetooth/BluetoothManager.html)，然后用于获取适配器。Android 4.3（API等级18）介绍 [BluetoothManager](https://developer.android.com/reference/android/bluetooth/BluetoothManager.html)：

      ```java
      private BluetoothAdapter mBluetoothAdapter;
      ...
      // Initializes Bluetooth adapter.
      final BluetoothManager bluetoothManager =
              (BluetoothManager) getSystemService(Context.BLUETOOTH_SERVICE);
      mBluetoothAdapter = bluetoothManager.getAdapter();
      ```

2. 启用蓝牙

  接下来，您需要确保蓝牙已启用。调用[isEnabled()](https://developer.android.com/reference/android/bluetooth/BluetoothAdapter.html#isEnabled())来检查蓝牙是否已启用。如果此方法返回`false`，则蓝牙被禁用。以下片段检查是否启用了蓝牙。如果不是，该片段会显示一个错误，提示用户转到设置以启用蓝牙：

  ```java
  // 确保蓝牙在设备上可用并且已启用。如果没有
  // 显示一个请求用户许可的对话框来启用蓝牙。
  if (mBluetoothAdapter == null || !mBluetoothAdapter.isEnabled()) {
      Intent enableBtIntent = new Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE);
      startActivityForResult(enableBtIntent, REQUEST_ENABLE_BT);
  }
  ```
  **注意：**`REQUEST_ENABLE_BT`传递给的常量[startActivityForResult(android.content.Intent, int)](https://developer.android.com/reference/android/app/Activity.html#startActivityForResult(android.content.Intent, int))是一个本地定义的整数（它必须大于0），系统在[onActivityResult(int, int, android.content.Intent)](https://developer.android.com/reference/android/app/Activity.html#onActivityResult(int, int, android.content.Intent))实现中将其作为 `requestCode`参数传回给您。

## 查找BLE设备

要找到BLE设备，请使用该 [startLeScan()](https://developer.android.com/reference/android/bluetooth/BluetoothAdapter.html#startLeScan(android.bluetooth.BluetoothAdapter.LeScanCallback))方法。这个方法需要 [BluetoothAdapter.LeScanCallback](https://developer.android.com/reference/android/bluetooth/BluetoothAdapter.LeScanCallback.html) 一个参数。您必须实现此回调，因为这是如何返回扫描结果。由于扫描是电池密集型的，因此应遵守以下准则：

- 一旦找到所需的设备，请停止扫描。
- 不要在循环中进行扫描, 并且在扫描时设置其超时时间. 之前可用的设备可能已经离开了可用范围,继续扫描将会浪费电池电量.

以下代码展示了如何启动和停止扫描：

```java
/**
 * 用于扫描和显示可用BLE设备的活动。
 */
public class DeviceScanActivity extends ListActivity {

    private BluetoothAdapter mBluetoothAdapter;
    private boolean mScanning;
    private Handler mHandler;

    // 10s后停止扫描
    private static final long SCAN_PERIOD = 10000;
    ...
    private void scanLeDevice(final boolean enable) {
        if (enable) {
            // 在预定义的扫描周期后停止扫描。
            mHandler.postDelayed(new Runnable() {
                @Override
                public void run() {
                    mScanning = false;
                    mBluetoothAdapter.stopLeScan(mLeScanCallback);
                }
            }, SCAN_PERIOD);

            mScanning = true;
            mBluetoothAdapter.startLeScan(mLeScanCallback);
        } else {
            mScanning = false;
            mBluetoothAdapter.stopLeScan(mLeScanCallback);
        }
        ...
    }
...
}
```

如果您只想扫描特定类型的外设，则可以调用[startLeScan(UUID[], BluetoothAdapter.LeScanCallback)](https://developer.android.com/reference/android/bluetooth/BluetoothAdapter.html#startLeScan(android.bluetooth.BluetoothAdapter.LeScanCallback)), 通过`UUID[]`指定您的应用程序支持的GATT服务的数组。

以下是[BluetoothAdapter.LeScanCallback](https://developer.android.com/reference/android/bluetooth/BluetoothAdapter.LeScanCallback.html)用于提供BLE扫描结果的界面的一个实现 ：

```java
private LeDeviceListAdapter mLeDeviceListAdapter;
...
// 设备扫描回调.
private BluetoothAdapter.LeScanCallback mLeScanCallback =
        new BluetoothAdapter.LeScanCallback() {
    @Override
    public void onLeScan(final BluetoothDevice device, int rssi,
            byte[] scanRecord) {
        runOnUiThread(new Runnable() {
           @Override
           public void run() {
               mLeDeviceListAdapter.addDevice(device);
               mLeDeviceListAdapter.notifyDataSetChanged();
           }
       });
   }
};
```

**注意：**您只能扫描**蓝牙LE设备**或扫描**经典蓝牙设备**，如 [蓝牙](https://developer.android.com/guide/topics/connectivity/bluetooth.html)中所述。您无法同时扫描蓝牙LE和经典设备。

## 连接到GATT服务器

与BLE设备交互的第一步是连接到它，更具体地说，连接到设备上的GATT服务器。要连接到BLE设备上的GATT服务器，请使用该 [connectGatt()](https://developer.android.com/reference/android/bluetooth/BluetoothDevice.html#connectGatt(android.content.Context, boolean, android.bluetooth.BluetoothGattCallback))方法。该方法有三个参数：一个[Context](https://developer.android.com/reference/android/content/Context.html)对象 `autoConnect`（布尔值，指示是否在BLE器件变为可用时自动连接）以及对以下参数的引用 [BluetoothGattCallback](https://developer.android.com/reference/android/bluetooth/BluetoothGattCallback.html)：

```java
mBluetoothGatt = device.connectGatt(this, false, mGattCallback);
```

这连接到由BLE设备托管的GATT服务器，并返回一个[BluetoothGatt](https://developer.android.com/reference/android/bluetooth/BluetoothGatt.html)实例，然后您可以使用该 实例来执行GATT客户端操作。调用者（Android应用程序）是GATT客户端。将[BluetoothGattCallback](https://developer.android.com/reference/android/bluetooth/BluetoothGattCallback.html)用于提供结果到客户端，如连接状态，以及任何进一步的GATT客户端操作。

在这个例子中，BLE应用程序提供了一个activity（`DeviceControlActivity`）来连接，显示数据，并显示设备支持的GATT服务和特性。根据用户的输入，这个活动与一个[Service](https://developer.android.com/reference/android/app/Service.html)被调用的通信， `BluetoothLeService`通过Android BLE API与BLE设备交互：

```java
// 通过Android BLE API与BLE设备交互的服务。.
public class BluetoothLeService extends Service {
    private final static String TAG = BluetoothLeService.class.getSimpleName();

    private BluetoothManager mBluetoothManager;
    private BluetoothAdapter mBluetoothAdapter;
    private String mBluetoothDeviceAddress;
    private BluetoothGatt mBluetoothGatt;
    private int mConnectionState = STATE_DISCONNECTED;

    private static final int STATE_DISCONNECTED = 0;
    private static final int STATE_CONNECTING = 1;
    private static final int STATE_CONNECTED = 2;

    public final static String ACTION_GATT_CONNECTED =
            "com.example.bluetooth.le.ACTION_GATT_CONNECTED";
    public final static String ACTION_GATT_DISCONNECTED =
            "com.example.bluetooth.le.ACTION_GATT_DISCONNECTED";
    public final static String ACTION_GATT_SERVICES_DISCOVERED =
            "com.example.bluetooth.le.ACTION_GATT_SERVICES_DISCOVERED";
    public final static String ACTION_DATA_AVAILABLE =
            "com.example.bluetooth.le.ACTION_DATA_AVAILABLE";
    public final static String EXTRA_DATA =
            "com.example.bluetooth.le.EXTRA_DATA";

    public final static UUID UUID_HEART_RATE_MEASUREMENT =
            UUID.fromString(SampleGattAttributes.HEART_RATE_MEASUREMENT);

    // 由BLE API定义的各种回调方法.
    private final BluetoothGattCallback mGattCallback =
            new BluetoothGattCallback() {
        @Override
        public void onConnectionStateChange(BluetoothGatt gatt, int status,
                int newState) {
            String intentAction;
            if (newState == BluetoothProfile.STATE_CONNECTED) {
                intentAction = ACTION_GATT_CONNECTED;
                mConnectionState = STATE_CONNECTED;
                broadcastUpdate(intentAction);
                Log.i(TAG, "Connected to GATT server.");
                Log.i(TAG, "Attempting to start service discovery:" +
                        mBluetoothGatt.discoverServices());

            } else if (newState == BluetoothProfile.STATE_DISCONNECTED) {
                intentAction = ACTION_GATT_DISCONNECTED;
                mConnectionState = STATE_DISCONNECTED;
                Log.i(TAG, "Disconnected from GATT server.");
                broadcastUpdate(intentAction);
            }
        }

        @Override
        // 发现新设备
        public void onServicesDiscovered(BluetoothGatt gatt, int status) {
            if (status == BluetoothGatt.GATT_SUCCESS) {
                broadcastUpdate(ACTION_GATT_SERVICES_DISCOVERED);
            } else {
                Log.w(TAG, "onServicesDiscovered received: " + status);
            }
        }

        @Override
        // 特征读取操作的结果
        public void onCharacteristicRead(BluetoothGatt gatt,
                BluetoothGattCharacteristic characteristic,
                int status) {
            if (status == BluetoothGatt.GATT_SUCCESS) {
                broadcastUpdate(ACTION_DATA_AVAILABLE, characteristic);
            }
        }
     ...
    };
...
}
```

当一个特定的回调被触发时，它会调用适当的 `broadcastUpdate()`帮助器方法并传递一个动作。请注意，本节中的数据解析是根据蓝牙心率测量 [配置文件规范执行的](http://developer.bluetooth.org/gatt/characteristics/Pages/CharacteristicViewer.aspx?u=org.bluetooth.characteristic.heart_rate_measurement.xml)：

```java
private void broadcastUpdate(final String action) {
    final Intent intent = new Intent(action);
    sendBroadcast(intent);
}

private void broadcastUpdate(final String action,
                             final BluetoothGattCharacteristic characteristic) {
    final Intent intent = new Intent(action);

    // 这是心率测量配置文件的特殊处理。 
    // 数据解析按照配置文件规范进行。
    if (UUID_HEART_RATE_MEASUREMENT.equals(characteristic.getUuid())) {
        int flag = characteristic.getProperties();
        int format = -1;
        if ((flag & 0x01) != 0) {
            format = BluetoothGattCharacteristic.FORMAT_UINT16;
            Log.d(TAG, "Heart rate format UINT16.");
        } else {
            format = BluetoothGattCharacteristic.FORMAT_UINT8;
            Log.d(TAG, "Heart rate format UINT8.");
        }
        final int heartRate = characteristic.getIntValue(format, 1);
        Log.d(TAG, String.format("Received heart rate: %d", heartRate));
        intent.putExtra(EXTRA_DATA, String.valueOf(heartRate));
    } else {
        // For all other profiles, writes the data formatted in HEX.
        final byte[] data = characteristic.getValue();
        if (data != null && data.length > 0) {
            final StringBuilder stringBuilder = new StringBuilder(data.length);
            for(byte byteChar : data)
                stringBuilder.append(String.format("%02X ", byteChar));
            intent.putExtra(EXTRA_DATA, new String(data) + "\n" +
                    stringBuilder.toString());
        }
    }
    sendBroadcast(intent);
}
```

回来`DeviceControlActivity`，这些事件由[BroadcastReceiver](https://developer.android.com/reference/android/content/BroadcastReceiver.html)处理：

```java
// 处理由服务发起的各种事件。
// ACTION_GATT_CONNECTED: 连接到GATT服务器.
// ACTION_GATT_DISCONNECTED: 与GATT服务器断开连接.
// ACTION_GATT_SERVICES_DISCOVERED: 发现GATT服务.
// ACTION_DATA_AVAILABLE: 从设备接收数据. 这可能是读取或通知操作的结果。
private final BroadcastReceiver mGattUpdateReceiver = new BroadcastReceiver() {
    @Override
    public void onReceive(Context context, Intent intent) {
        final String action = intent.getAction();
        if (BluetoothLeService.ACTION_GATT_CONNECTED.equals(action)) {
            mConnected = true;
            updateConnectionState(R.string.connected);
            invalidateOptionsMenu();
        } else if (BluetoothLeService.ACTION_GATT_DISCONNECTED.equals(action)) {
            mConnected = false;
            updateConnectionState(R.string.disconnected);
            invalidateOptionsMenu();
            clearUI();
        } else if (BluetoothLeService.
                ACTION_GATT_SERVICES_DISCOVERED.equals(action)) {
            // 在用户界面上显示所有支持的服务和特性。
            displayGattServices(mBluetoothLeService.getSupportedGattServices());
        } else if (BluetoothLeService.ACTION_DATA_AVAILABLE.equals(action)) {
            displayData(intent.getStringExtra(BluetoothLeService.EXTRA_DATA));
        }
    }
};
```

## 读取BLE属性

一旦你的Android应用程序连接到GATT服务器并发现服务，它就可以在支持的地方读取和写入属性。 例如，这段代码会遍历服务器的服务和特性，并在UI中显示它们：

```java
public class DeviceControlActivity extends Activity {
    ...
    // 演示如何遍历支持的GATT服务/特性。
    // 在本示例中，我们填充绑定到UI上的ExpandableListView的数据结构。
    private void displayGattServices(List<BluetoothGattService> gattServices) {
        if (gattServices == null) return;
        String uuid = null;
        String unknownServiceString = getResources().
                getString(R.string.unknown_service);
        String unknownCharaString = getResources().
                getString(R.string.unknown_characteristic);
        ArrayList<HashMap<String, String>> gattServiceData =
                new ArrayList<HashMap<String, String>>();
        ArrayList<ArrayList<HashMap<String, String>>> gattCharacteristicData
                = new ArrayList<ArrayList<HashMap<String, String>>>();
        mGattCharacteristics =
                new ArrayList<ArrayList<BluetoothGattCharacteristic>>();

        // 循环遍历可用的GATT服务。
        for (BluetoothGattService gattService : gattServices) {
            HashMap<String, String> currentServiceData =
                    new HashMap<String, String>();
            uuid = gattService.getUuid().toString();
            currentServiceData.put(
                    LIST_NAME, SampleGattAttributes.
                            lookup(uuid, unknownServiceString));
            currentServiceData.put(LIST_UUID, uuid);
            gattServiceData.add(currentServiceData);

            ArrayList<HashMap<String, String>> gattCharacteristicGroupData =
                    new ArrayList<HashMap<String, String>>();
            List<BluetoothGattCharacteristic> gattCharacteristics =
                    gattService.getCharacteristics();
            ArrayList<BluetoothGattCharacteristic> charas =
                    new ArrayList<BluetoothGattCharacteristic>();
           // 循环遍历可用的特性。
            for (BluetoothGattCharacteristic gattCharacteristic :
                    gattCharacteristics) {
                charas.add(gattCharacteristic);
                HashMap<String, String> currentCharaData =
                        new HashMap<String, String>();
                uuid = gattCharacteristic.getUuid().toString();
                currentCharaData.put(
                        LIST_NAME, SampleGattAttributes.lookup(uuid,
                                unknownCharaString));
                currentCharaData.put(LIST_UUID, uuid);
                gattCharacteristicGroupData.add(currentCharaData);
            }
            mGattCharacteristics.add(charas);
            gattCharacteristicData.add(gattCharacteristicGroupData);
         }
    ...
    }
...
}
```

## 接收GATT 通知(Notifications)

BLE应用程序在发生特定特性变化时要求收到通知是很常见的。这段代码展示了如何使用[setCharacteristicNotification()](https://developer.android.com/reference/android/bluetooth/BluetoothGatt.html#setCharacteristicNotification(android.bluetooth.BluetoothGattCharacteristic, boolean))方法为特性设置通知:

```java
private BluetoothGatt mBluetoothGatt;
BluetoothGattCharacteristic characteristic;
boolean enabled;
...
mBluetoothGatt.setCharacteristicNotification(characteristic, enabled);
...
BluetoothGattDescriptor descriptor = characteristic.getDescriptor(
        UUID.fromString(SampleGattAttributes.CLIENT_CHARACTERISTIC_CONFIG));
descriptor.setValue(BluetoothGattDescriptor.ENABLE_NOTIFICATION_VALUE);
mBluetoothGatt.writeDescriptor(descriptor);
```

一旦为特征启用了通知，[onCharacteristicChanged()](https://developer.android.com/reference/android/bluetooth/BluetoothGattCallback.html#onCharacteristicChanged(android.bluetooth.BluetoothGatt, android.bluetooth.BluetoothGattCharacteristic)) 如果远程设备上的特性发生更改，则会触发回调：

```java
@Override
// Characteristic notification
public void onCharacteristicChanged(BluetoothGatt gatt,
        BluetoothGattCharacteristic characteristic) {
    broadcastUpdate(ACTION_DATA_AVAILABLE, characteristic);
}
```

## 关闭客户端应用程序

一旦您的应用程序完成使用BLE设备，它应该调用， [close()](https://developer.android.com/reference/android/bluetooth/BluetoothGatt.html#close()) 以便系统可以适当地释放资源

```java
public void close() {
    if (mBluetoothGatt == null) {
        return;
    }
    mBluetoothGatt.close();
    mBluetoothGatt = null;
}
```

## 相关示例

- [Android BluetoothAdvertisements Sample](https://github.com/googlesamples/android-BluetoothAdvertisements/)

  演示如何使用Bluetooth Low Energy API广播少量数据。 还演示如何扫描这些广播。 （需要2个设备才能看到完整的操作）

- [Android BluetoothLeGatt Sample](https://github.com/googlesamples/android-BluetoothLeGatt/)

  本示例演示如何使用Bluetooth LE通用属性配置文件（GATT）在设备之间传输任意数据。

- [DevBytes: Bluetooth Low Energy API in Android 4.3](https://www.youtube.com/watch?v=vUbFB1Qypg8)

  Android BLE APi演示视频

