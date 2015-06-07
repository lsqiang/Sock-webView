# Socket初探
socket原始方式获取网络数据展现到webView<br>

##一、方法参数介绍

>     /**1.socket
>      参数
>      domain:  协议域，AF_INET(IPV4的网络开发)
>      type:    Socket 类型，SOCK_STREAM(TCP)/SOCK_DGRAM(UDP,报文)
>      protocol:IPPROTO_TCP,协议，如果输入0，可以根据第二个参数，自动选择协议
>      
>      返回值
>      socket,如果 > 0就表示成功
>     */
>     int clientSocket = socket(AF_INET, SOCK_STREAM, 0);
> 
> 
>     /**2.connect
>      参数
>      1> 客户端socket
>      2> 指向数据结构sockaddr的指针，其中包括目的端口和IP地址
>      服务器的"结构体"地址
>      提示：C 语言中没有对象
>      
>      3> 结构体数据长度
>      0 成功/其他 错误代号，非0即真
>      
>      返回值
>      */
>     struct sockaddr_in serverAddress;
>     // 1> 协议族
>     serverAddress.sin_family = AF_INET;
>     // 2> ip 找机器 inet_addr 会对地址做字节翻转
>     serverAddress.sin_addr.s_addr = inet_addr("127.0.0.1");
>     // 3> 端口找程序，将整数的高低位互换(字节翻转)
>     serverAddress.sin_port = htons(12345);
>     int result = connect(clientSocket, (const struct sockaddr *)&serverAddress, sizeof(serverAddress));
>     
>     if (result == 0) {
>         NSLog(@"成功");
>     } else {
>         NSLog(@"失败");
>     }
> 
>     /**3.发送
>      参数
>      1> 客户端socket
>      2> 发送内容地址 void * == id
>      3> 发送内容长度 => 字节长度
>      4> 发送方式标志，一般为0
>      
>      返回值
>      如果成功，则返回发送的字节数，失败则返回SOCKET_ERROR
>     */
>     NSString *msg = @"约？";
>     ssize_t sendLen = send(clientSocket, msg.UTF8String, strlen(msg.UTF8String), 0);
>     NSLog(@"发送了 %ld %tu %ld", sendLen, msg.length, strlen(msg.UTF8String));
>     
>     /**4.接收数据
>      参数
>      1> socket
>      2> 接收内容的地址
>      3> 长度
>      4> 接收标志，如果是0，标示阻塞式，一直等待服务器的返回数据
>      
>      C语言中，数组的名字，就是指向数组第一个元素的指针
>      
>      返回值
>      接收数据的长度
>      */
>     uint8_t buffer[1024];
>     
>     ssize_t recvLen = recv(clientSocket, buffer, sizeof(buffer), 0);
>     NSLog(@"接收 %ld 字节", recvLen);
>     
>     // 获取服务器返回的二进制数据
>     NSData *data = [NSData dataWithBytes:buffer length:recvLen];
>     // 转换成字符串
>     NSString *str = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
>     NSLog(@"%@", str);
>     
>     //5.断开连接
>     close(clientSocket);

    
##二、加载显示到webView上

> - (void)viewDidLoad {
>     [super viewDidLoad];
>     
>     //1.connect
>     if (![self connect:@"124.167.222.38" port:80]) {
>         return;
>     }
>     
>     //2.send
>     NSString *request = @"GET / HTTP/1.1\r\n"
>             "Host: m.qidian.com\r\n"
>             "User-Agent: iPhone AppleWebKit\r\n"
>             "Connection: Close\r\n\r\n";
>     
>     NSString *result = [self sendAndReceive:request];
>     
>     NSRange range = [result rangeOfString:@"\r\n\r\n"];
>     if (range.location != NSNotFound) {
>         NSString *html = [result substringFromIndex:range.location];
>         
>         [self.webView loadHTMLString:html baseURL:[NSURL URLWithString:@"http://m.qidian.com"]];
>         
>         NSLog(@"%@", html);
>     } else {
>         
>         NSLog(@"error");
>     } }

##三、封装方法
###1、封装连接方法
//1.连接

> - (BOOL)connect:(NSString *)address port:(int)port {
>     
>     //1.socket
>     self.clientSocket = socket(AF_INET, SOCK_STREAM, 0);
>     
>     //2.connect
>     struct sockaddr_in serverAddress;
>     
>     serverAddress.sin_family = AF_INET;
>     serverAddress.sin_addr.s_addr = inet_addr(address.UTF8String);
>     serverAddress.sin_port = htons(port);
>     
>     int result = connect(self.clientSocket, (const struct sockaddr *)&serverAddress, sizeof(serverAddress));
>     
>     if (result == 0) {
>         return YES;
>     }
>     return NO; }

###2、封装发送接收消息方法

> - (NSString *)sendAndReceive:(NSString *)msg {
>     
>     send(self.clientSocket, msg.UTF8String, strlen(msg.UTF8String), 0);
>     
>     // 获取服务器返回的二进制数据
>     uint8_t buffer[1024];
>     NSMutableData *dataM = [NSMutableData data];
>     ssize_t recvLen = -1;
>     while (recvLen != 0) {
>         // 循环接收数据
>         recvLen = recv(self.clientSocket, buffer, sizeof(buffer), 0);
>         
>         // 将数据拼接到 dataM 中
>         [dataM appendBytes:buffer length:recvLen];
>     }
>     // 转换成字符串
>     NSString *str = [[NSString alloc] initWithData:dataM encoding:NSUTF8StringEncoding];
>     
>     return str; }

###3、断开连接
//3.断开连接

> - (void)disconnection {
>     // 断开连接
>     close(self.clientSocket); }



