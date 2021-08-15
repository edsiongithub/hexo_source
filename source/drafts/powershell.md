### powershell执行策略错误
win10中执行powershell脚本出现如下错误：
```
Activate.ps1，因为在此系统上禁止运行脚本。有关详细信息，请参阅 https:/go.microsoft.com/fwlink/?LinkID=135170 中的 about_Execution_Policies。
```
powershell的执行策略问题，使用get-executionpolicy查看策略

```
Restricted 执行策略不允许任何脚本运行。  

AllSigned 和 RemoteSigned 执行策略可防止 Windows PowerShell 运行没有数字签名的脚本。

 本主题说明如何运行所选未签名脚本（即使在执行策略为 RemoteSigned 的情况下），还说明如何对  脚本进行签名以便您自己使用。

有关 Windows PowerShell 执行策略的详细信息，请参阅 about_Execution_Policy。
```
使用set-executionpolicy设置策略，以管理员身份启动powershell，执行以下命令： ``` set-executionpolicy RemoteSigned```
