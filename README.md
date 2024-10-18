# 什么是Modbus
   - Modbus是一种应用层协议，用于在工业控制领域进行通信。
   - <img src="https://pic1.zhimg.com/70/v2-89bf76c87e200cfaaa2532e6a91dbeba_1440w.avis?source=172ae18b&biz_tag=Post">
   - Modicon公司于1979年制定了Modbus协议标准，并用在其PLC产品上。后来Modicon公司被施耐德收购。已成为一种事实标准协议，同时也被IEC-61158工业通信总线规范收录于type 15子集
   - 
# Modbus版本
   - Modbus-RTU: 单工模式，适用于串口通信，速度快，但只能在线路上进行通信，不支持广播。
   - Modbus-ASCII: ASCII编码，适用于串口通信，速度快，但只能在线路上进行通信，不支持广播。
   - Modbus-TCP: TCP/IP协议，适用于网络通信，速度慢，支持广播。
   - Modbus-UDP: UDP协议，适用于网络通信，速度快，支持广播。   
   - Modbus-RTU和Modbus-ASCII是串口通信协议，Modbus-TCP和Modbus-UDP是网络通信协议。   
  
# Modbu标准
   - Modbus-串口版本基本定义了物理层、链路层以及应用层
   - <img src="https://pica.zhimg.com/80/v2-2e51d99d7bf8426b986e866f4a0a538e_1440w.webp">

# 链路层
1. 单播与广播
   - modbus从链路控制的角度属于主(Master)/从(Slave)方式，比较简单。对介质的访问控制相当于时分复用。通讯总是由主站发起，但可分为单播和广播两种方式，单播就是主站向特定的从站发出通讯请求，广播是向总线所有的设备发起通讯请求。看下面两个图就比较清楚了：
   1. 单播：报文中的地址字段指定所需要访问的设备，该设备收到请求后作出对应的应答
      <img src="https://pic2.zhimg.com/80/v2-c15d83fd3fa3c5b137258f776aa19bc1_1440w.webp">
   2. 广播(Broadcast)：主站向总线所有设备发出广播报文，所有从设备都不做应答，报文中地址为0则为广播请求
      <img src="https://pica.zhimg.com/80/v2-f3b0507e7c3e4b4c4d925b3f8a323944_1440w.webp"> 
2. 寻址‍
   - modbus-RTU从设备都具有一个单字节地址，其地址分配定义为：
     - 地址0：广播地址，所有的从设备必须处理广播报文。
     - 1-247：从设备地址，主设备是没有地址的，这一点需要注意。
     - 248-255：保留地址
     <img src="https://pic1.zhimg.com/80/v2-5dff464e0104240560de9a25ab4949aa_1440w.webp">
3. 报文结构
   - 通信模式是主/从方式，也即主请求、从应答的方式。无论主请求报文，还是从应答报文其结构都是如下图这样的
   - <img src="https://pic1.zhimg.com/80/v2-fb3de6ba1091bb428c892473083ad06a_1440w.webp">
   1. 地址：取值范围是0-247，如果是0，就是主站广播报文；如果是1-247，则有可能是主站请求或者从站应答。
   2. 功能码：也就是报文命令，代表主站对从站的操作，读或者写
   3. 数据：数据字段，主请求报文，从应答报文会有所差异。也就是说假设抓取总线报文，如何区分是主站请求还是从站应答，则需要通过数据字段进行区分了。
   4. CRC校验：采样CRC16，16位循环冗余校验
4. 主站状态机
   - 链路层一个最重要的职责就是对通讯介质的管理，如果没有介质的管理，就不能成其为总线。Modbus如何进行介质管理呢？
   - 前面介绍了从链路管理的角度来看，总线介质上发送报文的有两种设备，一种是主设备，另一种是从设备。
   - 对于主设备来说，它会有两种报文会向总线介质发送：一种是广播报文，另一种是单播报文。那么究竟是怎么控制介质的呢？其实很简单，看看下面这个状态机就很清楚了
   - <Img src="https://pic2.zhimg.com/80/v2-9e16c8913cd71a4ee2ce47fb4818fa49_1440w.webp">
   - 图中的事件的产生，将会触发主设备链路状态机从一个状态迁移到另一个状态，再事件触发后，还伴随动作需要执行。
     - 空闲：主设备处于空闲态。如果此时应用程序需要发送一个从设备请求，就会切换到等待应答状态；如果此时应用需要发送总线广播，此时，主设备就切换到广播延时。
     - 等待应答：在等待应答状态时，主设备将等待来自从设备的应答报文，如果接收到从站的报文，则进入应答处理。如果等待超时则进入错误处理状态机；在等待过程中，状态机不发生迁移。
     - 广播延时：如果主设备需要发送广播报文，则发送完就进入广播延时状态。这里为什么要延时呢？延时的设计目的就是留给从设备一点时间去处理接收到的广播请求。如果主设备没有这个延时，那么如果应用马上在发一个请求，则从设备有可能来不及处理。但是从设备只做接收处理，任何从设备都不可以对广播报文进行应答
5. 从站状态机
    - 对于从设备来说，只接收主设备请求或者发送应答，因此从设备的状态机就更简单了 
    - <img src="https://pic3.zhimg.com/80/v2-5c94e4768edcd1c4596915a297d69de2_1440w.webp">
    - 从设备的状态机很简单，系统一上电就进入空闲状态，空闲态一直监听总线报文，当收到一个完整的报文时，首先校验报文的正确性，再检查报文是否是发给该设备的，如果是请求本设备的，则先完成请求的操作，然后准备好应答报文，如果出错则将出错信息发送给主站。如果收到的是主站广播请求，则仅仅处理相应请求，不做任何应答
  6. 介质管理
     - 对于帧的时间管理，其实就是对介质的冲突管理，modbus-RTU对于介质管理规定了2个重要的时间参数，以实现成帧、冲突管理等。来看看下面这几个图 
       - <img src="https://pic1.zhimg.com/80/v2-a0df7ea7b0e42aef4b91baccc55fd6b6_1440w.webp">
       - ​这个图可以用于断帧，也就时判断是否接收到一个完整的帧，因此只需要使用一个定时器在每次收到一个字节后，就重启一个3.5字节定时器，如果这个3.5字节定时器中断了，就证明收到了一个Modbus报文，至于这个报文是不是正确的报文，可以在进一步根据帧格式进行校验
     - 另外还规定了报文需要连续发送，字节间隔不得超过1.5字节时间
       - <img src="https://pic3.zhimg.com/80/v2-4e8cc419bfeba5a3baf6eb16ab498d5c_1440w.webp">
     - 上面对于介质管理所规定得时分复用，可以用一个状态机来描述
       - <img src="https://pica.zhimg.com/80/v2-a9cbc33c5d1a5a327a57a02a992b8cea_1440w.webp">
  7. 应用层
     1. Client/Server模型
        - 应用层通信可以看成是client/server方式，或许会与前面说得主从模式搅合在一起，导致理解起来费劲。其实这里client/server是从应用得角度描述得，modbus-RTU中，主设备其应用层就是client侧，而slave设备就是应用的server。modbus标准文档有种把简单问题复杂描述之嫌。其实就是这样一个简单的图
        - <img src="https://pica.zhimg.com/80/v2-04cf50296581c671b7d072fa1b4867f4_1440w.webp">
        - 无错误：Client(主站)向从站发出请求，Server(从站)执行命令请求的操作，然后发送应答给Client(主站)，这里的操作，有可能是读取参数，设置参数，或者执行某个动作，具体取决于产品怎么设计。
        - 有错误：Client(主站)向从站发出请求，Server(从站)检测到错误，然后发送异常应答给Client(主站)。这里的错误，有可能是读取失败，寄存器地址非法，写失败，执行动作失败等。
     2. 数据模型
        - Modbus将采用大端字节顺序传输报文，什么意思呢？比如一个16位数据0x55AA，先传输高字节0x55，再传输低字节0xAA 
        - Modbus将数据抽象成四张表：
          - <img src="https://pica.zhimg.com/80/v2-507af5f5440d5b6d4001bce46f86cf52_1440w.webp">
          - 线圈:modbus协议原本是Modicon公司针对其PLC产品开发的协议，与其特殊的工业PLC控制编程有很大的关系。作为使用modbus协议进行应用开发而言，则不必费力研究为什么叫这些名字,理解为开关量即可
          - 这四个表本质上就是将应用数据规划为离散位开关量，以及寄存器变量，其中线圈与保持寄存器表为可读可写，其他两个表为只读。这个四个表中将应用数据都利用寄存器地址进行索引。地址范围为0x0000-0xFFFF。需要理解的是，这里的地址与芯片的地址空间完全是两个概念，把它简单理解成modbus可以索引0x0000-0xFFFF这么多个用户应用16位数据即可。其中有的可能是开关量，有的可能利用两个连续寄存器对应用户的浮点数，字符串等等，都有可能
          - 这些地址是modbus请求命令中的一个字段，比如使用的最最频繁的两条命令，就是0x03,0x10两条命令
          - modbus标准中的0x03号命令的请求以及应答定义
            - <img src="https://pica.zhimg.com/80/v2-26d7ed20f66cf7eabca9c9b7e8fc1a32_1440w.webp">
          - 又比如0x10号命令
            - <img src="https://pic4.zhimg.com/80/v2-16d0a84b4df352632b76694109fcb9b5_1440w.webp">
# Modbus命令
1. modbus-RTU支持的命令或者叫操作码，如下面这个表
   - <img src="https://pica.zhimg.com/80/v2-dd14ff548cb13c4d7a19c1eecc3df210_1440w.webp">
   - 其中最为常用的命令是0x03,0x04,0x06,0x10号命令，一般的应用而言，单个位开关量通信效率不免低下，现在很多产品开发已很少使用。其实对于这样的离散量也完全可以直接放在输入寄存器表以及保持寄存器表中。modbus对于用户应用并没有严格的规定。用户可以自由进行寄存器地址(或叫索引) 映射
## 查询功能码0x03
   主机发送: 01 03 0000 0001 84 0A
            -- -- ---- ---- -----
            |  |    |     |   crc16
            |  |    |     要读取寄存器个数
            |  |    起始寄存器地址
            |  功能码
            设备地址
            <img src="https://i-blog.csdnimg.cn/blog_migrate/135ce122ad96e8ac3c065f794581df14.png">
  从机回复: 01 03 02 19 98 B2 7E 
           -- -- -- ----- -----
            |  |    |     crc16
            |  |    返回的寄存器数据
            |  功能码
            设备地址
            <img src="https://i-blog.csdnimg.cn/blog_migrate/48ff30b2a9213548522776f2f3ff3fde.png">
## 写入功能码0x06(单个寄存器)
   主机发送: 01 06 00 00 00 01 48 0A
            -- -- ----- ----- -----
            |  |    |     |   crc16
            |  |    |     待写入的数据
            |  |    要写入的寄存器地址
            |  功能码
            设备地址
   从机回复: 01 06 00 00 00 01 48 0A
## 写入功能码0x10(多个寄存器)
   主机发送起始地址+寄存器个数+总字节数+数据，从机返回起始地址+寄存器数量

# modbuscrc16
```c
unsigned short do_crc(unsigned char *ptr, int len)
{
    unsigned int i;
    unsigned short crc = 0xFFFF;
    
    while(len--)
    {
        crc ^= *ptr++;
        for (i = 0; i < 8; ++i)
        {
            if (crc & 1)
                crc = (crc >> 1) ^ 0xA001;
            else
                crc = (crc >> 1);
        }
    }
    
    return crc;
}
int crc16 = do_crc(buf, sizeof(buf));
```
# 大彩屏modbus的使用
1. 安装VisualHMI_V1.1.359
2. 打开demo: 1.HMI80480KM070
3. 点“编译运行”，会打开虚拟机,同时会打开一个调试窗体，选择串口"COM2",并打开
4. 打开串口调试助手，选择“COM1“，可以看到虚拟机输出的信息

5. 关闭虚拟机，把协议调整成modbus,并(点教本编程)修改lua程序如下:
```lua
--数据类型定义
VT_LW = 1    --变量地址	--界面上控件的变量地址
VT_RW = 2    --FLASH存储
VT_0x = 10    --线圈
VT_1x = 11    --输入点
VT_3x = 12    --输入寄存器
VT_4x = 13    --保持寄存器    --modbus寄存器

--系统定时回调(一个系统周期调用一次，回调速度手工程复杂程度影响。最快20ms)
function on_run(screen)
	local val1 = get_uint16(VT_4x,  0x0001)	-- 把modbus的数据(VT_4x)同步到界面控件1001上
	local val2 = get_uint16(VT_LW,  0x1001)
	if val1 ~= val2 then
		set_uint16(VT_LW,  0x1001, val1) 
	end

--变量变化回调函数
function on_update(slave,vtype,addr)
	if addr == 0x1001 then	--把控件1001的数据同步到modbus上
		set_uint16(VT_4x, 0x0001, val)
	elseif addr == 0x1002 then
		set_uint16(VT_4x, 0x0002, val)
		print("set vt_4x"..val)
	end
	
```
