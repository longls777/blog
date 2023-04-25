---

title: python如何发送邮件
tags: 
categories: Practice
date: 2023-04-25 21:19:00
index_img: 
banner_img: 
---



**这里记录如何在python中发送邮件，可以将模型运行结果实时发送至自己的邮箱查看**



## 首先获取自己邮箱的SMTP账号密码等信息

#### 登录qq邮箱

![image-20230425202106261](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230425202106261.png)



#### 进入帮助中心

![image-20230425202013243](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230425202013243.png)

#### 搜索授权码

![image-20230425202130751](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230425202130751.png)



#### 选择第一个

![image-20230425202141582](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230425202141582.png)



****



#### 点击账号与安全

![image-20230425202207812](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230425202207812.png)





#### 点击安全设置，选择生成授权码（SMTP服务要先开启）

![image-20230425202236608](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230425202236608.png)



#### 根据提示获取授权码，保留等待以后使用

![image-20230425202255138](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230425202255138.png)



## 发送邮件

#### qq邮箱的SMTP发送服务器相关设置如下（如有更新请查看最新规则）

![image-20230425202555587](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230425202555587.png)

#### 以下是相关代码

```python
import smtplib
from email.mime.multipart import MIMEMultipart 
from email.mime.text import MIMEText
from email.header import Header


def send_email(machine, model_name, message):
    # 连接邮箱服务器
    address = "smtp.qq.com" # 发送邮件服务器地址
    port = 465 # 端口
    mail = smtplib.SMTP_SSL(address, port) # 获取邮箱对象
    # 登陆邮箱
    user = "1111111111@qq.com" # 你的qq邮箱的完整邮件地址
    password = "xxxxxxxxxxx" # 前面获得的授权码
    mail.login(user, password)
    
    email = MIMEMultipart()
    subject = Header(machine + '--' + model_name, 'utf-8').encode() # 第一个参数是邮件主题，自己设置
    email['Subject'] = subject
    email['From'] = '11111111@qq.com <11111111@qq.com>' # 发送人，写自己的qq邮箱地址（不知道能不能写别的，可以自己尝试）
    email['To'] = '222222222@bupt.cn' # 接收人邮箱地址
    text = MIMEText(message, 'plain', 'utf-8') 

    email.attach(text)
    mail.sendmail('111111111@qq.com','2222222222@bupt.cn', email.as_string()) # 发送邮箱和接收邮箱
    mail.quit()

if __name__ == "__main__":
    send_email("双卡3090", "model16", "test acc : 79.2%")
```



#### 结果

![image-20230425211821914](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230425211821914.png)





> 参考 https://zhuanlan.zhihu.com/p/560555163