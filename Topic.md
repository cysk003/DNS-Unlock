将用户信息存入mysql 并 通过（"接口将数据写入R2 Bucket 保证数据安全不丢失）

会员状态信息在登录时通过mysql查询 查询后 向 mongoDB发送会员状态，同步会员日志，记录用户习惯

mysql通过卡密来确定会员购买情况

邮件系统 方便 用户 找回账号密码

卡密(每次生成50卡密(uuid) 当卡密剩余10个时向邮件发出告警)卡密用完即失效同时该用户成为会员

用户反馈机制 谷歌问卷/群组

