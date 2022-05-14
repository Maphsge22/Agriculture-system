# 1.系统设计与实现
本设计的是“智慧农业信息采集控制系统”，系统主要实现的功能有：

（1）采集终端向服务器上报温度、湿度、电机状态（用于降低温度）、开关状态（用于自动浇水）；

（2）服务器接收到数据后提取信息，将数据及其上报时间写入数据库存储历史记录，便于用于查看；

（3）服务器接收到数据后从数据库中读取用户设定的阈值，对数据进行判断，如果超过或者低于阈值即发送对应指令，打开/关闭电机，或者打开/关闭开关；

（4）用户端程序可以修改报警阈值；

整个系统的实现架构如下图：

（1）农业采集终端运行在树莓派上（在桌面Linux上也可以运行）；

（2）农业采集系统服务端使用云服务器，操作系统为Ubuntu20.04；

（3）数据存储使用MySQL服务器；

（4）农业采集用户端运行在桌面Linux上，操作系统为Ubuntu16.04；

![image-20220512210608988](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220512210608988.png)

其中，所有数据均采用JSON数据格式，使用UDP传输，数据库中有两张表，一张表为history用于保存历史数据，一张表为value用于保存阈值信息。

# 2. 建立数据库及数据表

① 建立数据库ia_system:

```sql
create database ia_system;
```

② 建立数据表history:

```sql
create table history (id INT(10) not null,temp FLOAT,humi FLOAT,motor CHAR(3), switch CHAR(3));
```

③ 建立数据表value:

```sql
create table value (id INT(10) not null,temp_max FLOAT,humi_max FLOAT,temp_min FLOAT,humi_min FLOAT)
```

④ 提前在数据表value中设置一个阈值：

```sql
insert into value(id,temp_max,humi_max,temp_min,humi_min) values(2019212617,40,40,10,10);
insert into value(id,temp_max,humi_max,temp_min,humi_min) values(2019212615,50,50,0,0);
```

![image-20220512210653685](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220512210653685.png)

![image-20220512212151473](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220512212151473.png)

![image-20220512210717174](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220512210717174.png)

# 3. 运行服务端

服务端需要在云服务器（项目使用腾讯云轻量应用服务器，操作系统为ubuntu20.04）上运行，使用gcc编译。

进入到server文件夹，执行make命令编译：

```bash
cd server
make
```

![image-20220512210856230](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220512210856230.png)

然后运行程序（默认监听8002端口，使用UDP协议，如果有安全组需要放行该端口）:
```bash
./server
```

# 4. 运行客户端

客户端运行在桌面Linux（使用VMware+ubuntu16.04操作系统）上，进入client文件夹后使用make命令编译：

```bash
cd client
make
```

![image-20220512211611983](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220512211611983.png)

编译之后运行程序：

```bash
./client
```

## 4.1. 查询当前阈值

![image-20220512211646429](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220512211646429.png)

## 4.2. 修改阈值

修改最大湿度值为50.5：
![image-20220512211721413](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220512211721413.png)

## 4.3. 更换终端

![image-20220512212027073](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220512212027073.png)

# 5. 运行终端

终端中只使用到了UDP Socket编程，所以可以编译为桌面Linux的程序，也可以编译为ARM开发板（树莓派）上的程序。

进入endpoint文件夹，编译：
```bash
cd endpoint
make endpoint_PC
```
![image-20220512212348553](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220512212348553.png)

运行程序，第一个参数是云服务器ip，第二个参数是云服务器端口：

![image-20220512221649633](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220512221649633.png)

当前模拟的是终端每隔1min向服务器上报一次数据，下一步研究可以考虑使用传感器获取真实参数。在服务端也可以看到上报的数据：

![image-20220512221505885](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220512221505885.png)

在MySQL中查看历史记录：

![image-20220512221707876](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220512221707876.png)

