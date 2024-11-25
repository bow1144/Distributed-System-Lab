# 实验四、Java-RMI 实验报告

## 1. RMI 应用文件列表及放置情况

### **文件列表**
1. 远程接口文件: `someInterface.java`
2. 远程接口实现文件: `SomeImpl.java`
3. 服务器端程序: `SomeServer.java`
4. 客户端程序: `SomeClient.java`
5. Stub 文件: `SomeImpl_Stub.class` (由 `rmic` 生成)

### **文件放置情况**
```
project/
├── Server/
│   ├── someInterface.java
│   ├── someInterface.class
│   ├── SomeImpl.java
│   ├── SomeImpl.class
│   ├── SomeImpl_Stub.class
│   ├── SomeServer.java
│   ├── SomeServer.class

├── Client/
│   ├── someInterface.class
│   ├── SomeImpl_Stub.class
|   ├── SomeClient.java
|   ├── SomeClient.class
```

---

## 2. 每个文件主要实现的功能和文件相互间的关系

### **文件作用**
1. `someInterface.java`:
   - 定义远程接口，含有远程调用方法
   - 作为 Stub 和服务器实现的通信协议

2. `SomeImpl.java`:
   - 实现远程接口，包含远程调用的具体逻辑
   - Stub 依赖此实现

3. `SomeServer.java`:
   - 创建服务器程序，将远程对象注册到 RMI 注册表

4. `SomeClient.java`:
   - 通过 RMI 注册表获取服务器远程对象，调用其方法

5. `SomeImpl_Stub.class`:
   - Stub 文件，将客户端的调用转成服务器评测调用

### **文件相互间的关系**
- 客户端通过 `Stub` 和 RMI 注册表连接服务器
- 服务器通过实现接口方法，返回客户端请求

---

## 3. 每个文件的伪代码

### **someInterface.java**
```plaintext
// 定义接口
声明远程方法: sayHello(name: String) 返回 String
```

### **SomeImpl.java**
```plaintext
// 实现 someInterface
进行通信初始化
实现 sayHello(name: String): 返回 "Hello, " + name
```

### **SomeServer.java**
```plaintext
// 创建实现对象
创建注册表
创建实现对象
将实现对象添加到 RMI 注册表
```

### **SomeClient.java**
```plaintext
// 通过 RMI 注册表获取对象
获取注册表中服务器地址
调用 sayHello(name: String)
输出服务器返回的结果
```

---

## 4. 实验运行情况和结果

### **实验运行步骤**
* 首先完成服务器端的开发、编译
* 再完成客户端的开发、编译
* 连接成功
<img width="609" alt="{DE3AF147-3EFE-4579-A480-AAD527CE29C3}" src="https://github.com/user-attachments/assets/88461a9a-68cb-4871-9cc9-b13ab4b8a18d">

### **实验中遇到的问题
1. 使用`javac SomeClient.java`编译时，无法识别汉字
   - 原因：IDEA特有的默认设置为GBK
   - 解决方法：使用`javac -encoding UTF-8 SomeClient.java`
2. `@Override`报错：无法定位重载符
   - 解决方法：先写完父类再写子类
3. *java：...包不存在*
   - 原因：maven没配置好
   - 解决方法：配置好就行了
