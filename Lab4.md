# 分布式主机监控系统

## 一、实验准备与环境
* Java 8 IDEA
* SpringBoot，Socket Servlet
* HTML/CSS/JSP
* 本实验的网站设计基于第三个实验（Web2）

## 二、客户端设计
客户端需要实现的是网站的的客户端，其中需要实现的功能是：

* 获取本机的资源（包括CPU，内存等信息），登录时间
* 客户端设计了一个打卡按钮，可以向服务端传输打卡时间

public class Local {

    // 获取本地系统的设备名称（设备名称一般可以自定义，这里以"Client_Device_01"为示例）
    public String getDeviceName() {
        // 获取本地机器的 InetAddress
        try {
            InetAddress inetAddress = InetAddress.getLocalHost();
            return inetAddress.getHostName();  // 返回本机的IP地址
        } catch (IOException e) {
            e.printStackTrace();
            return "Unknown Device Name";  // 如果获取失败，返回"Unknown IP"
        }
    }

    // 获取本地IP地址
    public String getLocalIPAddress() {
        try {
            InetAddress inetAddress = InetAddress.getLocalHost();
            return inetAddress.getHostAddress();  // 返回本机的IP地址
        } catch (IOException e) {
            e.printStackTrace();
            return "Unknown IP";  // 如果获取失败，返回"Unknown IP"
        }
    }

      // 获取设备状态（在线状态示例）
      public String getDeviceStatus() {
          return "Online";  // 默认在线
      }
  
      // 获取CPU使用率
      public double getCPUUsage() {
          OperatingSystemMXBean osBean = (OperatingSystemMXBean) ManagementFactory.getOperatingSystemMXBean();
          // 获取系统的 CPU 负载
          double cpuLoad = osBean.getSystemCpuLoad();
  
          // 如果返回 -1，表示无法获取 CPU 使用率
          if (cpuLoad == -1) {
              System.out.println("无法获取 CPU 使用率");
              return -1;
          }
  
          // 转换为百分比并返回
          return cpuLoad * 100;
      }
  
  
      // 获取内存使用率（百分比）
      public double getMemoryUsage() {
          OperatingSystemMXBean osBean = (OperatingSystemMXBean) ManagementFactory.getOperatingSystemMXBean();
          long totalMemory = osBean.getTotalPhysicalMemorySize();  // 总内存
          long freeMemory = osBean.getFreePhysicalMemorySize();   // 剩余内存
          return (1 - (freeMemory / (double) totalMemory)) * 100;  // 计算内存使用率
      }
  
      // 获取当前系统时间的时间戳（单位：毫秒）
      public long getLoginTime() {
          return System.currentTimeMillis();  // 获取当前时间的时间戳
      }
  
      // 获取CPU型号
      public String getCpuModel() {
          String cpuModel = "Unknown";
          try {
              Process process = Runtime.getRuntime().exec("wmic cpu get caption");
              BufferedReader reader = new BufferedReader(new InputStreamReader(process.getInputStream()));
              cpuModel = reader.readLine();  // 获取输出中的 CPU 型号
          } catch (Exception e) {
              e.printStackTrace();
          }
          return cpuModel;
      }

## 三、客户端向服务端的传输
1. 使用Local类获取本机状态
2. 通过sendDeviceInfo方法将设备信息发送给服务器：首先使用JSONObject构建包含设备信息的JSON对象；构造成功后设置JSON类型的请求头，将获取的信息发送给服务器

  public class localInformation {
  
      public static void main(String[] args) {
          // 获取本地系统信息（这里只是一个示例，实际可以通过使用API获取更多信息）
          Local localInfo = new Local();
  
          // 获取本地系统信息
          String deviceName = localInfo.getDeviceName();
          String ipAddress = localInfo.getLocalIPAddress();
          String status = localInfo.getDeviceStatus();
          double cpuUsage = localInfo.getCPUUsage();
          double memoryUsage = localInfo.getMemoryUsage();
          String cpuModel = localInfo.getCpuModel();
          String osVersion = localInfo.getOSVersion();
          // 将设备信息发送到服务器
          sendDeviceInfo(deviceName, ipAddress, status, cpuUsage, memoryUsage, cpuModel, osVersion);
      }
  
      public static void sendDeviceInfo(String deviceName, String ipAddress, String status,
                                        double cpuUsage, double memoryUsage, String cpuModel, String osVersion) {
          try {
              // 构建JSON对象
              JSONObject json = new JSONObject();
              json.put("deviceName", deviceName);
              json.put("ipAddress", ipAddress);
              json.put("status", status);
              json.put("cpuUsage", cpuUsage);
              json.put("memoryUsage", memoryUsage);
              json.put("loginTime", System.currentTimeMillis());
              json.put("cpuModel", cpuModel);
              json.put("osVersion", osVersion);
  
              System.out.println(json.toString());
  
              // 发送POST请求
              URL url = new URL("http://localhost:8080/monitor/updateStatus");
              HttpURLConnection connection = (HttpURLConnection) url.openConnection();
              connection.setRequestMethod("POST");
              connection.setDoOutput(true);
              connection.setRequestProperty("Content-Type", "application/json");
  
              try (OutputStream os = connection.getOutputStream()) {
                  byte[] input = json.toString().getBytes("utf-8");
                  os.write(input, 0, input.length);
              }
  
              // 获取响应
              int responseCode = connection.getResponseCode();
              System.out.println("Response Code: " + responseCode);
  
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
  }
