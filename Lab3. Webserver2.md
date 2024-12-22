# 实现一个Web应用程序（2）

## 一、实验工具与环境
* Java8， IDEA
* SpringBoot
* MySQL
* HTML/CSS/JSP

## 二、实验概述
本实验的目的是在Web应用程序中连接数据库，使其拥有增删查改的功能。本实验使用了类似学生教务系统的数据，通过前端三件套、JPA数据库连接完成项目

## 三、后端内容
### 3.1 Spring Boot的后端结构
1. 控制层controller
2. 服务层service
3. 数据访问层repository
4. 实体类model

### 3.2 控制层
控制层负责接收客户端的HTTP请求，调用Service层处理业务逻辑，控制层的功能如下：

1. 获取所有用户的信息，展示到网页上
2. 编辑用户信息的按钮
3. 增加用户的按钮
4. 删除用户的按钮（逻辑在编辑信息之下）

public class UserController {

    @Autowired
    private UserService userService;

    @Autowired
    private MonitorService monitorService;

    // 获取所有用户
    @GetMapping()
    public String getAllUsers(Model model) {
        List<User> users = userService.getAllUsers();  // 获取用户数据
        System.out.println("Users data: " + users);
        model.addAttribute("user", users);  // 将用户数据传递给模板
        return "user";  // 返回视图名 "user"（指向 user.html 模板）
    }

    @GetMapping("/editUser")
    public String editUser(@RequestParam("id") Long userId, Model model) {
        // 根据用户ID获取用户信息（从数据库或服务中查询）
        System.out.println("EditUser endpoint hit with ID: " + userId);
        User user = userService.getUserById(userId);
        System.out.println("User data: " + user);
        // 将用户信息添加到模型中
        model.addAttribute("user", user);
        // 返回编辑页面
        return "editUser";
    }

    @PostMapping("/updateUser")
    public String updateUser(@ModelAttribute User user) {
        // 调用服务层更新用户信息
        userService.updateUser(user);
        // 重定向到用户列表页面
        return "redirect:/users";
    }

    @GetMapping("/addUser")
    public String addUserPage(Model model) {
        // 创建一个空的 User 对象用于表单绑定
        model.addAttribute("user", new User());
        return "addUser"; // 显示新增用户页面 (addUser.html)
    }

    @PostMapping("/addUser")
    public String addUser(@ModelAttribute User user) {
        // 打印接收到的用户信息，调试用
        System.out.println("Received User Data: " + user);

        // TODO: 调用服务层保存用户
        userService.addUser(user);

        // 重定向到用户列表页面
        return "redirect:/users";
    }

    // 删除用户
    @PostMapping("/deleteUser")
    @ResponseBody  // 表示返回的不是视图，而是响应的内容（JSON）
    public String deleteUser(@RequestParam("id") Long userId) {
        try {
            userService.deleteUser(userId);  // 调用服务层删除用户
            return "redirect:/users";  // 返回成功标识
        } catch (Exception e) {
            return "error";  // 返回失败标识
        }
    }

    // 查询用户详情
    @GetMapping("/details")
    public String detailsPage(@RequestParam("id") Long userId, Model model) {
        List<LocalDateTime> attendance = userService.getDetail(userId);
        User user = userService.getUserById(userId);

        Map<LocalDate, List<String>> attendanceByDate = attendanceService.groupAttendanceByDate(attendance);
        Map<LocalDate, List<Boolean>> validityMap = attendanceService.checkAttendanceValidity(attendance);

        model.addAttribute("attendance", attendance);
        model.addAttribute("user", user);
        model.addAttribute("attendanceByDate", attendanceByDate);
        model.addAttribute("validityMap", validityMap);
        model.addAttribute("userId", userId);

        return "userDetail";
    }
}

### 3.3 服务层
服务层包含具体的用户（在这个业务中与控制层一一对应）以及与数据库连接

@Service
public class UserService {

    @Autowired
    private UserMapper userMapper;

    // 获取所有用户
    public List<User> getAllUsers() {
        return userMapper.getAllUsers();
    }

    public User getUserById(Long userId) {
        return userMapper.getUserById(userId);
    }

    public void updateUser(User user) {
        if (user.getId() == null) {
            throw new IllegalArgumentException("用户 ID 不能为空");
        }
        userMapper.updateUser(user);
    }

    public void addUser(User user) {
        if (user.getId() == null || user.getName() == null || user.getEmail() == null) {
            throw new IllegalArgumentException("id、用户名和邮箱不能为空");
        }
        for(User u : getAllUsers()) {
            if(user.getId().equals(u.getId())) throw new IllegalArgumentException("有重复id");
        }
        userMapper.addUser(user); // 调用 MyBatis 插入方法
    }

    // 删除用户的方法
    public void deleteUser(Long userId) {
        System.out.println("Try to delete id" + userId);
        userMapper.deleteUserById(userId);  // 调用 Mapper 删除用户
    }
}

### 3.4 数据访问层
通过MyBatis实现与MySQL的交互，本项目与数据库的交互包括  
1. 选择所有用户的数据
2. 修改用户的数据
3. 增加与删除用户

@Mapper
public interface UserMapper {

    // 查询所有用户
    // @Select("SELECT * FROM user")
    List<User> getAllUsers();

    User getUserById(@Param("id") Long id);

    void updateUser(User user);

    void addUser(User user);

    void deleteUserById(Long id);
}

### 3.5 数据库操作层
本项目的数据库操作使用mybatis-Mapper连接，用SQL语言操作数据库

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.test.mapper.UserMapper">

    <!-- 查询所有用户 -->
    <select id="getAllUsers" resultType="com.test.domain.User">
        SELECT * FROM spring_test.users
    </select>

    <!-- 查询指定用户 -->
    <select id="getUserById" parameterType="java.lang.Long" resultType="com.test.domain.User">
        SELECT * FROM spring_test.users WHERE id = #{id}
    </select>

    <!-- 更新用户信息 -->
    <update id="updateUser" parameterType="com.test.domain.User">
        UPDATE users
        SET
        name = #{name},
        email = #{email}
        WHERE id = #{id}
    </update>

    <!-- 插入新的用户 -->
    <insert id="addUser" parameterType="com.test.domain.User">
        INSERT INTO users (id, name, email)
        VALUES (#{id}, #{name}, #{email})
    </insert>

    <!-- 删除用户 -->
    <delete id="deleteUserById" parameterType="java.lang.Long">
        DELETE FROM spring_test.users WHERE id = #{id}
    </delete>
</mapper>

## 四、前端内容
    
    <!DOCTYPE html>
    <html lang="zh" xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>用户信息</title>
        <link rel="stylesheet" href="/css/styles.css">
    </head>
    <body>
    <div class="container">
        <div class="header">
            <h1 class="title">用户信息</h1>
            <!-- 添加和删除按钮容器 -->
            <div class="button-container">
                <button th:attr="onclick='addUser()'" class="action-btn add-user-btn">添加用户</button>
            </div>
        </div>
        <table class="table">
            <thead>
            <tr>
                <th>ID</th>
                <th>姓名</th>
                <th>Email</th>
                <th>操作</th>
            </tr>
            </thead>
            <tbody>
            <!-- 使用 Thymeleaf 渲染用户数据 -->
            <tr th:each="u : ${user}">
                <td th:text="${u.id}"></td>
                <td th:text="${u.name}"></td>
                <td th:text="${u.email}"></td>
                <td>
                    <!-- 修改按钮，点击后跳转到修改页面 -->
                    <button th:attr="onclick='editUser(' + ${u.id} + ')'">修改</button>
                    <button th:attr="onclick='viewDetails(' + ${u.id} + ')'" class="action-btn detail-btn">详情</button>
                </td>
            </tr>
            </tbody>
        </table>
    </div>
    
    <div class="sidebar">
        <h3>更多操作</h3>
        <a href="#" class="sidebar-btn" onclick="monitorHost()">监控主机</a>
        <a href="#" class="sidebar-btn" onclick="hostDetails()">主机列表</a>
        <a href="#" class="sidebar-btn" onclick="someOtherAction()">成果日志</a>
    </div>
    
    <!-- JavaScript -->
    <script>
        // 编辑用户的函数
        function editUser(userId) {
            // 跳转到编辑页面，并传递用户ID作为参数
            window.location.href = '/users/editUser?id=' + userId;
        }
    
        // 添加用户
        function addUser() {
            // 跳转到编辑页面，不传递参数
            window.location.href = '/users/addUser';
        }
    
        // 详细情况
        function viewDetails(userId) {
            // 跳转到详情页面
            window.location.href = '/users/details?id=' + userId;
        }
    
        // 监控主机
        function monitorHost() {
            // 跳转到主机监控页面
            window.location.href = '/monitor';
        }
    
        // 主机状态列表
        function hostDetails() {
            // 跳转到主机列表页面
            window.location.href = '/users/hostDetails';
        }
    </script>
    
    </body>
    </html>

## 五、实验效果

### 5.1 首页
<img width="644" alt="{DFA0FD1E-486E-4998-8A23-ADFFA5B868EE}" src="https://github.com/user-attachments/assets/d7143bd7-0df5-46ca-a480-316eec0da30f" />

### 5.2 增加用户
<img width="658" alt="{DEB8EE58-1094-4016-AF95-48716ECEA3DC}" src="https://github.com/user-attachments/assets/27e43b84-0d93-4576-84e4-5d3e5d257e4f" />

### 5.3 修改用户
<img width="614" alt="{E8035895-C8C0-43AC-9720-781961BF3318}" src="https://github.com/user-attachments/assets/d1543321-7a67-428c-ae02-5ed80fee3db0" />

### 5.3 删除用户
<img width="702" alt="{4992599F-0410-40B1-840F-FDC912BA7B6F}" src="https://github.com/user-attachments/assets/d05e17e5-9a37-4b80-bd44-1d08711a6577" />


## 五、实现中遇到的困难
1. maven配置：在配置maven是会出现有些依赖无法使用的Bug，解决方法是清除IDEA的缓存并重启IDEA
2. Spring Shell导致的SpringBoot的循环依赖，并且在禁用shell后依然无法解决；解决方法：在maven中取消shell的引用
3. Spring 报错 文档根元素 "mapper" 必须匹配 DOCTYPE 根 "null"：解决方法：将mapper文件放在main/resources.mapper下
