使用Zend Studio开发php应用程序的开始步骤2011年08月28日 星期日12:28 A.M.以下的步骤将会帮助你掌握使用zend studio开发和运行php应用程序的基本过程。
第一次打开zend studio，将会出现一个欢迎页面：
 
 
使用欢迎页面：
1. 浏览欢迎页面学习zend studio的特点和功能。在右上角点击“主页”图标，就能回到欢迎页面的主页。
2. 在欢迎页面，你可以点击图标来关闭欢迎页面。这时候zend studio的workbench（工作台）就会显示，并默认显示PHP视图。这个视图包含了一些画面来帮助你进行php开发。
PHP Explorer画面是一个文件系统画面，显示了在你的workspace（工作车间）内的php项目。
 
安装zend server（可选）
......（过程略）
 
创建一个php项目
为了开始编程，你必须创建一个PHP项目，来容纳你的应用程序文件。
为了创建一个PHP项目：
1. 从主菜单中，选择File | New | PHP Project 或者 在PHP Explorer 视图中，右键点击，然后选择New | PHP Project。 然后New PHP Project 指导页面将会出现。
 
 
2. 在项目名称框中输入项目的名字。
3. 在Contents 类目，选择新项目所在的位置和默认内容。选择项包括以下：
       o 在workspace（工作车间）内创建项目 —— 一个新的空项目将会在你的工作车间内创建。默认的当你第一次打开zend studio，一个车间将会出现在以下位置：@user.home/Zend/workspaces/DefaultWorkspace7
       o 从现在资源中创建一个项目 —— 创建一个项目，这个项目包含workspace目录之外的源文件。点击Browse 选择现有的文件。
       o 在本地服务器上创建一个项目 —— 在本地zend server上创建一个项目。这个选项只有当zend server已经安装好的情况下可选。
4. 点击Finish。
新的PHP项目将会被在你的workspace中创建，并在PHP Explorer视图中显示。查看Creating PHP Projects 来获得更多信息。
 
创建PHP文件
你现在可以通过在你的PHP项目中创建新的PHP文件来开始开发你的应用程序了。为了在你的PHP项目中创建一个新的PHP文件：
1. 在PHP Explorer视图，选定你创建好的项目。
2. 右键点击，然后选择New | PHP File 或者在主菜单中点击File 选择New | PHP File。创建新的PHP文件对话框将会出现：
 
 
3. 输入文件名，并点击Finish 完成创建文件。
你的文件将会在编辑器中打开，并在PHP Explorer视图中显示。
 
创建PHP元素
 
你可以使用Zend Studio PHP元素创建向导来迅速和简单地在你的PHP代码中创建类、接口等PHP元素。
为了创建一个新的PHP类或接口：
1. 在PHP Explorer 视图，右键点击你想创建新的元素的项目或是文件，选择New | Class 或是Interface。16:15 | 添加评论| 固定链接| 写入日志| phpeclipse for php 无法创建php project php项目
eclipse for php当创建一个新的php project的时候，输入完php项目名字，点击"next"却不能完成创建步骤。提示是：can not create php element，好像是这样的。原来这个是eclipse的一个bug，必须打上补丁。具体办法是：打开菜单栏的help => install new softwear，在安装新软件对话框中，work with 选择一个下载软件的站点，我的是：Galileo -http://download.eclipse.org/releases/galileo，等几秒钟，供安装的软件就会刷新出来，选择“PHP Development Tools (PDT) SDK Feature    2.1.2.v20090826-1027-7L7979F8NcJKhKRU19TmWA”和“PHP Development Tools (PDT) SDK Feature    2.1.1.v20090707-1108-51384QACJCQDWeNNmUTdPJewhS9O”这个两个软件安装，安装完毕，重启，应该就没有问题了。10:37 | 添加评论| 固定链接| 写入日志| php9月6日
Zend Studio 如何配置本地apache服务器使用xdebug调试php脚本
本地环境搭配：
apache 2.2 安装位置：D:\program files\Apache Software Foundation\Apache2.2
php 5.2.10 安装位置：C:\php
xdebug 已经安装并配置好
zend studio 安装位置：D:\program files\Zend\Zend Studio - 7.0.0
zend studio 默认workbench安装位置：C:\Users\Fred\Zend\workspaces\DefaultWorkspace7
 
配置apache，修改httpd.conf文件，在文件的最后添加：
 
Alias /Workspace "C:\Users\Fred\Zend\workspaces\DefaultWorkspace7"
<Directory "C:\Users\Fred\Zend\workspaces\DefaultWorkspace7">
Options Indexes MultiViews ExecCGI
DirectoryIndex index.php
AllowOverride None
Order allow,deny
Allow from all
*apache2.4
Require all granted
</Directory>
 
/Workspace这个名字随你喜欢改变
C:\Users\Fred\Zend\workspaces\DefaultWorkspace7这个和你的zend studio默认的workbench位置相对应
 
接下来配置zend studio。打开zend studio => Window => Preferences => PHP
PHP Executables => Add，如下所示设置（name可以随便起名字，但是之后的debug设置里面必须和这个name对应）：
 
 
PHP Servers　=> New，如下所示设置：
 
 
注意URL栏中添加了Workspace，这个是在httpd.conf中添加的目录别名。
 
PHP Debug，如下图所示，选择合适的Debugger，Server：
来源： http://www.2cto.com/kf/201202/118023.html
