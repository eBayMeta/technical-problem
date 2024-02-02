# 重现问题：配置Nginx反向代理的时候 如果代理目标地址是本服务器其他服务器的时候【也就是端口不同】
#         会存在本地环会问题，导致目标地址请求报错。

# 错误日志：

        cn.hutool.core.io.IORuntimeException: NoRouteToHostException: No route to host (Host unreachable)
            at cn.hutool.http.HttpRequest.send(HttpRequest.java:1098) ~[hutool-all-5.4.0.jar!/:na]
            at cn.hutool.http.HttpRequest.execute(HttpRequest.java:942) ~[hutool-all-5.4.0.jar!/:na]
            at cn.hutool.http.HttpRequest.execute(HttpRequest.java:913) ~[hutool-all-5.4.0.jar!/:na]
            at com.bot.task.CheckPayOrderService.check(CheckPayOrderService.java:51) ~[classes!/:0.0.1-SNAPSHOT]
            at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:na]
            at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:na]
            at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:na]
            at java.base/java.lang.reflect.Method.invoke(Method.java:566) ~[na:na]
            at org.springframework.scheduling.support.ScheduledMethodRunnable.run(ScheduledMethodRunnable.java:84) ~[spring-context-5.2.6.RELEASE.jar!/:5.2.6.RELEASE]
            at org.springframework.scheduling.support.DelegatingErrorHandlingRunnable.run(DelegatingErrorHandlingRunnable.java:54) ~[spring-context-5.2.6.RELEASE.jar!/:5.2.6.RELEASE]
            at org.springframework.scheduling.concurrent.ReschedulingRunnable.run(ReschedulingRunnable.java:93) ~[spring-context-5.2.6.RELEASE.jar!/:5.2.6.RELEASE]
            at java.base/java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:515) ~[na:na]
            at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:264) ~[na:na]
            at java.base/java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:304) ~[na:na]
            at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1128) ~[na:na]
            at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:628) ~[na:na]
            at java.base/java.lang.Thread.run(Thread.java:829) ~[na:na]
        Caused by: java.net.NoRouteToHostException: No route to host (Host unreachable)



# 问题本身：
        
        本地环回问题：通常涉及到一个设备尝试通过网络访问它自己提供的服务，这在使用域名系统（DNS）或特定的网络配置时尤为常见。
                    当一台设备（如服务器或个人电脑）上运行的客户端软件尝试通过网络（使用域名或公网 IP 地址）连接到同一设备上运行的服务器软件时，就可能遇到环回问题。
                    本地环回问题可能表现为连接失败或访问延迟，原因包括：
        
        DNS 解析：当设备尝试解析它自己的公网域名时，如果 DNS 解析指向了公网 IP 地址，
                 而设备位于 NAT（网络地址转换）后面或者没有适当的路由规则来处理这种"指向自己"的流量，就可能导致无法建立连接。
        
        路由配置：网络设备通常配置有路由规则，决定数据包应该如何在网络中转发。
                如果设备尝试通过其公网 IP 地址发送数据给自己，但网络配置不支持这种类型的流量"回环"到设备本身，就会导致连接问题。
        
        防火墙和安全策略：防火墙或安全策略可能会阻止从设备出去的流量再回到该设备上，特别是当流量看起来是从外部发起的时候。

# 解决方案：
        
        解决本地环回问题的一种常见方法是在设备的 hosts 文件中为目标域名添加一个条目，将域名直接解析到本地 IP 地址（如 127.0.0.1 或本地局域网地址）。
        这样，当设备尝试访问该域名时，它会直接通过本地网络接口进行，而不是通过外部网络，从而绕过了上述提到的问题。
        
        例如，在 Linux 或 macOS 系统中，可以编辑 /etc/hosts 文件；在 Windows 系统中，可以编辑 C:\Windows\System32\drivers\etc\hosts 文件，添加如下行：
        复制下面进行修改
        127.0.0.1   api.eshellpay.com

        这样，任何尝试访问 api.eshellpay.com 的请求都会被直接路由到本地机器，而不会尝试通过外部网络。

        
