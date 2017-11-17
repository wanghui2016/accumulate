
关键词

PowerShell查看当前版本、Windows版本、.NET版本信息

摘要

<p>PowerShell查看当前版本、Windows版本、.NET版本信息<br /><br />有很多cmdlet或者cmdlet的参数，在PowerShell不同的版本中，支持得不一样。所以，弄清楚当前PowerShell的版本信息是非常重要的一件事情。那么怎么查看当前PowerShell的版本信息呢？洪哥向大家介绍两个方法：<br /><br />其实就是两个PowerShell的环境变量，一个是$psversiontable，另一个是$host。<br /></p>
PowerShell查看当前版本、Windows版本、.NET版本信息

有很多cmdlet或者cmdlet的参数，在PowerShell不同的版本中，支持得不一样。所以，弄清楚当前PowerShell的版本信息是非常重要的一件事情。那么怎么查看当前PowerShell的版本信息呢？洪哥向大家介绍两个方法：

其实就是两个PowerShell的环境变量，一个是$psversiontable，另一个是$host。

先看看$psversiontable，这个变量拆开来看就是ps-version-table，表示PowerShell中各组件的版本号列表。其中表示PowerShell自己的版本号（PSVersion），也包括.NET的版本号（CLRVersion），还有Windows版本号（BuildVersion），其它的洪哥就不一一数了，其实洪哥也没有完全搞明白，呵呵。

```
PS C:\Users\zhanghong> $psversiontable

Name                           Value
----                           -----
CLRVersion                     2.0.50727.4984
BuildVersion                   6.1.7600.16385
PSVersion                      2.0
WSManStackVersion              2.0
PSCompatibleVersions           {1.0, 2.0}
SerializationVersion           1.1.0.1
PSRemotingProtocolVersion      2.1

接下来看看$host变量，里面一个Version，表示PowerShell的版本号。
PS C:\Users\zhanghong> $host

Name             : ConsoleHost
Version          : 2.0
InstanceId       : 38d7558e-1810-446d-a81c-41fb6d40ac13
UI               : System.Management.Automation.Internal.Host.InternalHostUserI
                  nterface
CurrentCulture   : zh-CN
CurrentUICulture : zh-CN
PrivateData      : Microsoft.PowerShell.ConsoleHost+ConsoleColorProxy
IsRunspacePushed : False
Runspace         : System.Management.Automation.Runspaces.LocalRunspace
```
