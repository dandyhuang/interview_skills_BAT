### dns本地服务器如何解析的

Linux服务器默认使用配置文件中指定的DNS服务器进行DNS查询。在大多数Linux发行版中，该配置文件为/etc/resolv.conf。该配置文件包含了一个或多个DNS服务器的IP地址，如果要在该文件中指定DNS服务器，可以使用以下命令：
sudo nano /etc/resolv.conf

在编辑器中添加以下行，其中<DNS服务器IP>为要添加的DNS服务器IP地址：
nameserver <DNS服务器IP>

可以添加多行来指定多个DNS服务器，例如：
nameserver 8.8.8.8
nameserver 8.8.4.4

上述配置将使用Google DNS服务器进行DNS查询。
如果在配置文件中没有指定DNS服务器，Linux服务器将使用默认的本地DNS服务器进行DNS查询。默认的本地DNS服务器通常由网络管理员或互联网服务提供商指定。