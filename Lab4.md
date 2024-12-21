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

## 四、服务器接收数据
* 服务器通过SpringBoot框架接收客户端数据，并进行JSON反序列化
* 由于服务器要得到状态更新时间，故将信息接收时间作为更新时间
* 将更新的时间用Java本地方法进行格式化

@Controller
@RequestMapping("/monitor")
public class MonitorController {

    @Autowired
    private MonitorService monitorService;

    @GetMapping()
    public String monitorPage(Model model) {
        List<HostDetails> details = monitorService.getAllDetails();
        // 格式化数据
        DecimalFormat decimalFormat = new DecimalFormat("#.0");
        for (HostDetails detail : details) {
            // 格式化 cpuUsage 和 memoryUsage
            detail.setCpuUsage(Double.parseDouble(decimalFormat.format(detail.getCpuUsage())));
            detail.setMemoryUsage(Double.parseDouble(decimalFormat.format(detail.getMemoryUsage())));
        }
        System.out.println("Details datas:" + details);
        model.addAttribute("details", details);
        return "monitor"; // 这里假设返回一个 Thymeleaf 模板
    }

    // 接收主机状态更新
    @PostMapping("/updateStatus")
    public ResponseEntity<String> updateDetails(@RequestBody HostDetails hostDetails) {
        // 确保 loginTime 是 LocalDateTime 类型
        String loginTime = LocalDateTime.now().toString();
        System.out.println("loginTime: " + loginTime);
        hostDetails.setLoginTime(loginTime);  // 设置转换后的时间

        System.out.println("Received host details: " + hostDetails);
        monitorService.saveHostDetails(hostDetails);
        return ResponseEntity.ok("Status updated successfully!");    }

    // 查询主机状态
    @GetMapping("/status/{hostname}")
    public HostStatus getHostStatus(@PathVariable String hostname) {
        return monitorService.getHostStatus(hostname);
    }
}

## 五、数据库操作：将客户端数据存入数据库
* 在数据库中实现

        <!-- 查询近期所有主机状态 -->
        <select id="insertHostDetails" resultType="com.test.domain.HostDetails">
            INSERT INTO host_details
            (device_name, ip_address, status, cpu_usage, memory_usage, login_time, cpu_model, os_version)
            VALUES (#{deviceName}, #{ipAddress}, #{status}, #{cpuUsage}, #{memoryUsage}, #{loginTime}, #{cpuModel}, #{osVersion})
        </select>
    
        <select id="getAllDetails" resultType="com.test.domain.HostDetails" resultMap="HostDetailsResultMap">
            SELECT device_name, ip_address, status, cpu_usage, memory_usage, login_time, cpu_model, os_version
            FROM spring_test.host_details
            ORDER BY id DESC
            LIMIT 10
        </select>
    
        <select id="getHostDetails" resultType="com.test.domain.HostDetails" resultMap="HostDetailsResultMap">
            SELECT
                t.id,
                t.device_name,
                t.ip_address,
                t.status,
                t.cpu_usage,
                t.memory_usage,
                t.login_time,
                t.last_updated,
                t.cpu_model,
                t.os_version
            FROM
                host_details t
            INNER JOIN (
                SELECT
                device_name,
                MAX(id) AS max_id
                FROM
                host_details
                GROUP BY
                device_name
            ) AS max_ids
            ON t.device_name = max_ids.device_name
            AND t.id = max_ids.max_id
            ORDER BY t.id DESC;  -- 按照 id 降序排列
        </select>


## 六、数据库设计与连接
<img width="589" alt="{C1206283-CB19-477B-9D35-1D1A063B36DC}" src="https://github.com/user-attachments/assets/9a5f26a7-622c-47b4-8ac0-1a5f63d68a5e" />

* 以自增id为主键，如果后期数据量过大的话可以考虑按年分表
* 值得注意的是SQL中的事件和Java的LocalDateTime的转换可能会有异常，需要在Java后端进行额外转换
* 更加值得注意的是，JPA和SQL之间应该是存在驼峰-下划线自动转化的，但在这个程序中莫名其妙地没用，所以只能在Mapper中要自定义映射，不然数据库加载会出错

        <resultMap id="HostDetailsResultMap" type="com.test.domain.HostDetails">
            <id property="id" column="id" />
            <result property="deviceName" column="device_name" />
            <result property="ipAddress" column="ip_address" />
            <result property="status" column="status" />
            <result property="cpuUsage" column="cpu_usage" />
            <result property="memoryUsage" column="memory_usage" />
            <result property="loginTime" column="login_time" />
            <result property="cpuModel" column="cpu_model" />
            <result property="osVersion" column="os_version" />
        </resultMap>

## 七、前端设计

    <!DOCTYPE html>
    <html lang="zh" xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>监控主机信息</title>
        <link rel="stylesheet" href="/css/styles.css">
    </head>
    <body>
    
    <div class="monitor-container">
        <div class="header">
            <h1 class="title">主机监控信息</h1>
            <div class="button-container">
                <!-- 按钮备用-->
            </div>
        </div>
        <table class="table">
            <thead>
            <tr>
                <th>主机名</th>
                <th>IP地址</th>
                <th>状态</th>
                <th>操作系统</th>
                <th>CPU型号</th>
                <th>CPU占用率</th>
                <th>内存占有率</th>
                <th>登录时间</th>
    
            </tr>
            </thead>
            <tbody>
            <!-- 使用 Thymeleaf 渲染用户数据 -->
            <tr th:each="d : ${details}">
                <td th:text="${d.deviceName}"></td>
                <td th:text="${d.ipAddress}"></td>
                <td th:text="${d.status}"></td>
                <td th:text="${d.osVersion}"></td>
                <td th:text="${d.cpuModel}"></td>
                <td th:text="${d.cpuUsage}"></td>
                <td th:text="${d.memoryUsage}"></td>
                <td th:text="${d.loginTime}"></td>
            </tr>
            </tbody>
        </table>
    </div>
    
    </body>
    </html>

## 八、问题与Bug
* Java客户端生成的使用率是小数点后十位小数，需要在后端规约为2位小数
* 数据库Mapper与后端之间的驼峰-下划线转换突然用不了，需要自定义映射才能正常运行
* 无法获取CPU利用率：暂时解决不了

## 九、结果展示
* 网页

<img width="906" alt="{B0493C33-DDDE-4190-8244-F43A69CE8D11}" src="https://github.com/user-attachments/assets/46889abf-2a9c-4366-9bfc-6a2a0c8bcca7" />

* 数据库

<img width="887" alt="{24BE44C0-6A8D-4F37-8E86-462710F4656E}" src="https://github.com/user-attachments/assets/2d39ed1a-7d65-4d17-a835-93a1705927dd" />

* 运行信息

<img width="1184" alt="{012D2A1D-228E-4AD6-A092-E555332BD15F}" src="https://github.com/user-attachments/assets/bc856ca3-a4da-45f4-b532-ff2b661bfcc7" />
