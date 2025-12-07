# Build Ladder
以下内容主要是用Trojan去代理本地流量去实现科学上网（简易版）
主要流程为 1.开机器 => 2.买域名申请ssl证书 =>3.上传证书 =>4.改配置 =>5.本地端配置
## 0.叠甲
以下内容仅用于个人学习或娱乐使用，不得用于违法等用途。

## 1.开一台阿里的ECS实例
去[阿里云官网](https://www.aliyun.com/)开一台最便宜的机器，服务器地址选香港的，镜像建议ubuntu系统的，流量宽带调高一点，宽带使用选“按使用流量”，登录凭证里选自定义密码，密码要**牢记**，其他一路默认就行。

## 2.购买域名，申请ssl证书
1. [阿里云万网](https://wanwang.aliyun.com/?utm_content=se_1018891736)随便购买一个域名
2. 完成实名认证
3. 然后在域名列表里点击**管理**找到**开启SSL证书**，去申请一个个人测试证书（免费）
4. 创建证书后下载证书（nginx服务器类型），后面要用
5. DNS解析：控制台 => 云解析DNS => 解析设置 => 添加记录
以上内容阿里云内都有详细文档说明

## 3.本地把下载的证书zip传到云端
本地cmd打开终端
把“C:\Users\user\Downloads\xxx_your_doamin_nginx.zip”改为下载的文件地址
“ecs-user”是ECS登录的用户名如果选择的是默认的“root”登录，那这里改为“root”，“公网地址”为购买的ECS的公网ip
```
scp "C:\Users\user\Downloads\xxx_your_doamin_nginx.zip" ecs-user@公网地址:/tmp/
```
输入后会要求你输入登录服务器的密码，这个密码是刚开始买ECS时自定义的密码


## 4.ESC里改Trojan配置
连接到ECS进去终端后，先更新一下
```
sudo apt update
sudo apt upgrade -y
```
下载Trojan和unzip
```
sudo apt install -y trojan
sudo apt install -y unzip
```
找到Trojan的配置文件，一般默认在`/etc/trojan/config.json`
我们需要把`.pem`和`.key`文件的目录填在配置文件`config.json`中
解压文件并移到指定位置
```
cd /tmp
unzip xxx_your_doamin_nginx.zip

# 解压后会返回两个文件，分别是证书文件和密钥文件
# 例如为your_cert_file.pem，your_private_file.key

sudo mkdir -p /etc/trojan/ssl/
sudo mv /tmp/your_cert_file.pem /etc/trojan/ssl/
sudo mv /tmp/your_private_file.key /etc/trojan/ssl/
```
用nano编辑`config.json`文件
```
sudo nano /etc/trojan/config.json
```
替换掉密码、证书路径、私钥路径
```
"password": [
        "您的安全强密码" 
    ],
    "ssl": {
        "cert": "/etc/trojan/ssl/your_cert_file.pem", 
        "key": "/etc/trojan/ssl/your_private_file.key",    
        // ...
    }
```
重启Trojan并配置自启动
```
sudo systemctl restart trojan
sudo systemctl enable trojan
```
检查Trojan的状态，确保输出状态为`active(running)`
```
sudo systemctl status trojan
```
**切记要在ECS的安全组里开放443端口**
云服务ECS => 网络与安全组 => 出方向 => 添加出方向规则
要把443端口放行  

完成以上内容云端就基本配置完成了
可以本地ping一下检查
cmd打开终端
```
ping your_domain
```
## 5.手机端或电脑端配置
手机端以shadowrocket为例：
1. 右上角添加 => 手动设置 => Trojan => 填写配置
2. 其中服务器地址写域名地址、端口443、密码填配置里写的
3. 传输协议tcp、伪装类型none、底层传输安全tls、SNI填域名地址、跳过证书验证false


