#安卓设备开发指导
#开发准备
####SDK发布库
AbleCloud发布的安卓设备SDK为`ac_device_android.jar`，除此之外，还需要导入`libDevice-Service.so`文件（可根据不同cpu做不同选择）

>**具体步骤**:把文件拷入你自己的工程的libs目录下并设置依赖
 
####开发环境设置
以下为 AbleCloud Android SDK 需要的所有的权限，请在你的AndroidManifest.xml文件里的`<manifest>`标签里添加
```java
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
<uses-permission android:name="android.permission.CHANGE_NETWORK_STATE"/>
```

####应用程序初始化
在你的应用使用AbleCloud服务之前，你需要在代码中对AbleCloud SDK进行初始化。

> 具体步骤:在启动App的`MainActivity`的`onCreate()`方法中调用此方法来进行初始化

开发阶段，请初始化测试环境
```java
/**
     * 请在主Activity的onCreate()中初始化安卓设备信息并开始连云操作
     *
     * @param mContext         获取Context实例
     * @param MajorDomainId    AbleCloud domainId，可到AbleCloud平台工程管理查看
     * @param SubDomainId      AbleCloud subDomainId，可到AbleCloud平台工程管理查看
     * @param physicalDeviceId AbleCloud设备物理ID，长度为16个字节，厂商需自己保证唯一性
     * @param secretKey        AbleCloud设备密钥，在产品管理中-->管理-->设备密钥，若使用AbleCloud默认分配的密钥对，则填写默认密钥里的私钥，如选择设备独立密钥入库，则需要使用密钥生成工具自己生成公私钥并把上传文件
     * @param version          AbleCloud设备版本，格式为"1-0-0";在初始化一个OTA版本后，若需要进行OTA升级，需要在设备管理中--         >OTA-->新建OTA版本把新的apk文件上传
     * @param mode             AC.TEST_MODE,当迁移到正式环境后使用AC.PRODUCTION_MODE，默认使用AC.PRODUCTION_MODE
     * @param regional         AbleCloud地域设置，AC.REGIONAL_CHINA代表中国地域，AC.REGIONAL_SOUTHEAST_ASIA代表东南亚地域，默认使用中国地域
     */
AC.init(Context Context, long MajorDomainId, long SubDomainId, String PhysicalDeviceId, String SecretKey, String Version, int Mode, int Regional);
```
**国内地域**
测试环境
```java
AC.init(this, MajorDomainId, SubDomainId, SecretKey, Version, AC.TEST_MODE);
```
在完成测试阶段之后，需要迁移到正式环境下。
```java
AC.init(this, MajorDomainId, SubDomainId, SecretKey, Version);
```
**国外地域**
```java
AC.init(this, MajorDomainId, SubDomainId, SecretKey, Version, AC.TEST_MODE, AC.REGIONAL_SOUTHEAST_ASIA);
```
在完成测试阶段之后，需要迁移到正式环境下。
```java
AC.init(this, MajorDomainId, SubDomainId, SecretKey, Version, AC.PRODUCTION_MODE, AC.REGIONAL_CHINA);
```
><font color=red>注</font>：初始化操作时厂商需要为每个设备分配一个**16字节长度的物理ID**，并保证该ID的唯一性。在厂商没有自己唯一的设备标识号情况下，建议使用**WIFI MAC地址**或者**手机IMEI号**并补0或其他将长度拼至16字节；可通过AbleCloud后台查看**在线设备**查看设备物理ID。

#交互消息
首先，我们从基础的数据结构开始。我们知道，安卓设备APP会与后端服务和普通app进行交互，因此AbleCloud定义了与云端的消息格式：

+ **ACDeviceMsg：**安卓设备与服务或普通app之间的交互消息，使用二进制或json或klv通讯协议。

##基本数据结构
####ACDeviceMsg
该消息用于处理安卓设备与服务之间的交互，框架会将ACDeviceMsg中的code部分解析出来，开发者可根据[code](firmware/wifi_interface_guide/#13 "消息码说明")来区分设备消息类型。但是ACDeviceMsg的payload部分由开发者解释，框架透传。ACDeviceMsg定义如下：
```java
public class ACDeviceMsg {

    /**
     * 消息码，用于区分消息类型
     * <p/>
     * 注意：64-200范围代表控制查询响应，200以上范围代表消息主动上报
     */
    private int msgCode;
    
    /**
     * 消息体，数据包payload，由开发者自己解析；
     * <p/>
     * 注意：若使用KLV格式消息进行交互，则通过getKLVObject方法获取KLVObject，其他格式则直接通过getPayload获取
     */
    private byte[] payload;

    /**
     * 消息下发类型
     * <p/>
     * CLOUD代表云端控制，DIRECT代表直连控制
     */
    private OpType type;
    
    //网关设备使用
    private ACSubDevice subDevice;
   
    //若使用通讯协议为KLV格式，则通过getKLVObject获取消息req
    public ACKLVObject getKLVObject() {}
    
    //若使用通讯协议为KLV格式，则通过setKLVObject设置resp（注意resp不需设置msgCode）
    public void setKLVObject(ACKLVObject object) {} 
	       
	//若使用通讯协议为二进制格式，则通过getPayload获取消息req
    public byte[] getPayload() {}
    
    //若使用通讯协议为二进制格式，则通过setPayload设置resp
    public void setPayload(byte[] payload) {}

    //若使用通讯协议为json格式，则通过getJsonPayload获取消息req
    public String getJsonPayload() {}

    //若使用通讯协议为json格式，则通过getJsonPayload设置resp（注意resp不需设置msgCode）
    public void setJsonPayload(String payload) {}

    /**
     * 判断消息下发控制对象
     *
     * @return true即为网关给子设备发送消息，false则为给网关自己发送消息
     */
    public boolean isSendToSubDevice() {}

    /**
     * 若消息下发对象为子设备，则通过此接口获取子设备信息
     *
     * @return ACSubDevice对象，具体定义如下所示
     */
    public ACSubDevice getSubDevice() {}
    
    //若为网关子设备上报消息，则需要调此接口设置子设备信息，其他情况下不需要调用
    public void setSubDevice(ACSubDevice subDevice) {
    
    public enum OpType {
        DIRECT,
        CLOUD
    }
}
```
从上面的定义可以看到，开发者需要根据code的不同值设置不同的resp

####ACKLVObject
ACKLVObject用于承载KLV数据格式交互的具体数据，我们称之为payload（负载）。
```java
public class ACKLVObject {
    private HashMap<Integer, Object> data = new HashMap<>();
   
    /**
     * 允许put参数的类型
     */
    private boolean checkType(Object o) {
        return  o == null ||
                o instanceof Boolean ||
                o instanceof Byte ||
                o instanceof Short ||
                o instanceof Integer ||
                o instanceof Long ||
                o instanceof Float ||
                o instanceof Double ||
                o instanceof String ||
                o instanceof byte[]
    }
    
	/**
     * 设置一个参数
     * @param key	参数名
     * @param <T>	参数值
     * @return
     */
    public <T> ACKLVObject put(String key, T value) {}

    /**
     * 获取一个参数值
     * @param key	参数名
     * @return		参数值
     */
    public <T> T get(String key) {}

    /**
     * 检查某一key是否存在
     * @param key	参数名
     * @return		存在返回true，否则返回false
     */
    public boolean contains(String key) {}

    /**
     * 获取所有的key值
     */
    public Set<String> getKeys() {}
}
```
><font color="brown">**注：**最常用的两个接口是put/get，若使用二进制或json的通讯协议，则不需要用到该类</font>

####ACSubDevice
该对象为网关子设备承载信息，独立设备则无需关心，具体接口定义如下：
```java
public class ACSubDevice {
    //子设备主域
    private long domainId;
    //子设备主域下所在子域
    private long subDomainId;
    //子设备物理Id(网关物理Id默认为4个0加mac地址)
    private String physicalDeviceId;

    public ACSubDevice(long domainId, long subDomainId, String physicalDeviceId) {}

    //getter
```

#SDK
##AC
前面，我们介绍了ablecloud SDK的交互消息，那到底具体该如何使用呢？SDK入口均通过AC来获取，简而言之，AC可以认为是SDK的框架，通过AC，开发者可以根据需要获取一系列服务、功能的接口。AC的定义如下：
```java
public class AC {

    /**
     * 调试模式
     */
    public static final int TEST_MODE = 1;
    public static final int PRODUCTION_MODE = 0;
    
    public static void init(Context context, long MajorDomainId, long SubDomainId, String secretKey, String version, int mode) {}

    /**
     * 获取AbleCloud物理Id，即4个0加上本机Mac地址,可用于生成二维码进行绑定
     *
     * @return 物理Id
     */
    public static String getPhysicalDeviceId() {}
    
    /**
     * 设置安卓设备连接状态的监听器
     * 注意：若多次设置监听，则只有最后一次设置有效，以下handleMsg接口也是如此；
     * 若提示消息或其他操作不涉及activity的其他元素，比如只需要显示toast，建议把它放在application里
     * 若有多个activity需要监听网络状态，则建议把它放在前一个activity的onResume函数里，当后一个activity返回时前一个activity依然生效
     *
     * @param listener sdk内部会自动进行断线重连，所以此处只需要取状态用于更新界面显示或提示用户
     */
    public static void setConnectListener(ACConnectChangeListener listener) {}
    
    /**
     * 处理Service-->安卓设备之间的交互消息
     *
     * @param handler 根据msgCode做不同处理并设置resp
     */
    public static void handleMsg(ACMsgHandler handler) {}
    
    /**
     * 处理网关子设备接入的交互消息
     *
     * @param handler 具体定义如下所示
     */
    public static void handleGatewayMsg(ACGatewayHandler handler) {}
    
    /**
     * 安卓设备主动上报数据到云端
     * 设备汇报的消息不需要响应。
     *
     * @param reqMsg 请求消息体
     * @throws Exception
     */
    public static void reportDeviceMsg(ACDeviceMsg reqMsg) {}
   
    /**
     * 在退出app的时候调用
     */
    public static void DeviceSleep() {
}
```

##消息处理接口Handler介绍

####ACConnectChangeListener
```java
public interface ACConnectChangeListener {
    
    /**
     * 设备连接上云端或者从云端断开连接时触发
     * 
     * 内置自动重连机制，开发者只需要可通过此接口提醒用户
     */
    void connect();

    void disconnect();
}
```

####ACMsgHandler
```java
public interface ACMsgHandler {

    /**
     * 处理Service-->安卓设备之间的交互消息
     *
     * @param reqMsg  请求消息体
     * @param respMsg 响应消息体
     */
    void handleMsg(ACDeviceMsg reqMsg, ACDeviceMsg respMsg);
}
```

####ACGatewayHandler
```java
public interface ACGatewayHandler {

    /**
     * 开启网关子设备接入
     *
     * @param timeout 超时时间
     * @throws Exception 开启失败即抛出异常
     */
    public void openGatewayMatch(long timeout) throws Exception;

    /**
     * 停止网关子设备接入
     * 
     * @throws Exception 停止失败即抛出异常
     */
    public void closeGatewayMatch() throws Exception;

    /**
     * 通知云端接入的所有子设备
     *
     * @param subDevices 所有接入的子设备列表（包括绑定或未绑定）
     */
    public void listSubDevices(List<ACSubDevice> subDevices);

    /**
     * 查询子设备是否在线
     *
     * @param subDevice 子设备
     */
    public boolean isSubDeviceOnline(ACSubDevice subDevice);

    /**
     * 剔除子设备（用户解绑子设备后，剔除该子设备接入）
     *
     * @param subDevice 解绑的子设备
     * @throws Exception 剔除失败即抛出异常
     */
    public void evictSubDevice(ACSubDevice subDevice) throws Exception;
}
```


