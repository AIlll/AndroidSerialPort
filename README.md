# AndroidSerialPort
Android 串口通信，基于[谷歌官方android-serialport-api](https://github.com/cepr/android-serialport-api)编译

![download](https://api.bintray.com/packages/aill/IOTMaven/AndroidSerialPort/images/download.svg)

### 使用说明

1. 在Module下的 build.gradle 中添加

```java
implementation 'com.aill:AndroidSerialPort:1.0.8'
```
2. 打开串口

```java
/**
 * @param 1 串口路径，根据你的设备不同，路径也不同
 * @param 2 波特率
 * @param 3 flags 给0就好
 */
SerialPort serialPort = new SerialPort(new File("/dev/ttyS1"), 9600, 0);
```
3. 往串口中写入数据

```java
//从串口对象中获取输出流
OutputStream outputStream = serialPort.getOutputStream();
//需要写入的数据
byte[] data = new byte[x];
data[0] = ...;
data[1] = ...;
data[x] = ...;
//写入数据
outputStream.write(data);
outputStream.flush();
```
4. 读取串口数据

读取数据时很可能会遇到分包的情况，即不能一次性读取正确的完整的数据

解决办法：可以在读取到数据时，让读取数据的线程sleep一段时间，等待数据全部接收完，再一次性读取出来。这样应该可以避免大部分的分包情况

```java
//从串口对象中获取输入流
InputStream inputStream = serialPort.getInputStream();
//使用循环读取数据，建议放到子线程去
while (true) {
    if (inputStream.available() > 0) {
        //当接收到数据时，sleep 500毫秒（sleep时间自己把握）
        Thread.sleep(500);
        //sleep过后，再读取数据，基本上都是完整的数据
        byte[] buffer = new byte[inputStream.available()];
        int size = inputStream.read(buffer);
    }
}
```

只接收一条数据的情况下，以上方法可以应对数据分包，数据量多的情况下需要考虑是否会因为sleep导致接收多条数据，可以根据通信协议核对包头包尾等参数。

5. 修改设备```su```路径

打开串口时，会检测读写权限，当没有权限时，会尝试对其进行提权

```java
//默认su路径是/system/bin/su，有些设备su路径是/system/xbin/su
//在new SerialPort();之前设置su路径
SerialPort.setSuPath("/system/xbin/su");
```

<br>

>- ByteUtil类：工具类，字符串转字节数组，字节数组转字符串
>- SerialFinder类：用于查找设备下所有串口路径
