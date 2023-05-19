- # 背景
	- 工作中需要用 android 真机调试 [[hippy]] 项目，每次都要执行 `adb reverse tcp:38989 tcp:38989` 有点麻烦
	- 手机还会时不时断开连接，重新插好又要来一次
	- 所以想要自动化这个工作
- # 初步尝试
	- 搜索了个脚本感觉很不错，才知道 [[adb]] 还有这种功能
		- https://stackoverflow.com/a/67572793/2202891
		- id:: 64677226-e8b5-41cb-85b0-c3bcd30615fe
		  ```
		  #!/bin/sh
		  while :
		  do
		      echo Waiting for Android USB device
		      adb wait-for-usb-device reverse tcp:38989 tcp:38989
		      echo Connection established
		      adb wait-for-usb-disconnect
		      echo Disconnected
		  done
		  ```
		- 阻塞直到设备连接，执行操作
			- `adb wait-for-usb-device xxx`
		- 阻塞直到设备断开连接
			- `adb wait-for-usb-disconnect`
	- 到这里已经实现目标了，只要不觉得一直开着一个黑黑的 terminal 麻烦就行，我接受不了。。。所以可以把它改造为 [[launchd]] Agent。
	- [[launchd]] 是 ((64677428-165b-4c3c-98c9-191a2c28028e))，可以 ((64677438-46a0-4536-a2df-d7edbac120ae))
	- 我们可以用它把前面的脚本，配置成登录后自动启动，没有界面，这种机制叫做 launchd agent，要了解更多可以参考[文档](https://www.launchd.info/)
	- 配置方式是编写 plist 配置文件，有钱也可以买 GUI 工具 [LaunchControl](https://www.soma-zone.com/LaunchControl/)
		- id:: 646775b4-2e11-470c-9543-36e0ed3aaeb2
		  ```
		  <?xml version="1.0" encoding="UTF-8"?>
		  <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
		  <plist version="1.0">
		  <dict>
		      <key>Label</key>
		      <string>com.tencent.cfanzhang.auto-adb-reverse</string>
		      <key>Program</key>
		      <string>/usr/local/bin/auto-adb-reverse</string>
		      <key>EnvironmentVariables</key>
		      <dict>
		          <key>PATH</key>
		          <string>/bin:/usr/bin:/usr/local/bin</string>
		      </dict>
		      <key>RunAtLoad</key>
		      <true/>
		      <key>StandardInPath</key>
		      <string>/tmp/auto-adb-reverse.log</string>
		      <key>StandardOutPath</key>
		      <string>/tmp/auto-adb-reverse.log</string>
		      <key>StandardErrorPath</key>
		      <string>/tmp/auto-adb-reverse.log</string>
		  </dict>
		  </plist>
		  
		  ```
	- [文档](https://www.launchd.info/)讲解的很好，我就不多解释了
- # 完整方案的步骤
	- 把这个文件保存到`~/Library/LaunchAgents/com.tencent.cfanzhang.auto-adb-reverse.plist`
		- ((646775b4-2e11-470c-9543-36e0ed3aaeb2))
		- **注意：**文件名必须和配置中的 Label 保持一致
		  background-color:: red
	- 把这个文件保存到 `/usr/local/bin/auto-adb-reverse`
		- ((64677226-e8b5-41cb-85b0-c3bcd30615fe))
		- 增加可执行权限`chmod +x /usr/local/bin/auto-adb-reverse`
	- 手动 load 这个 agent
		- ```
		  launchctl load ~/Library/LaunchAgents/com.tencent.cfanzhang.auto-adb-reverse.plist
		  ```
		- 我们也配置了 `RunAtLoad`，所以只要用户登陆了，都会自动加载。
	- 验证是否正常工作
		- 插上 android 机，执行 `adb reverse --list` 检查是否有列出 38989 端口
	- 查看日志
		- 配置文件中我们指定了 agent 把脚本的标准输入输出错误都记录到了 `/tmp/auto-adb-reverse.log` 文件，如果运行遇到问题可以查看这个文件的内容。