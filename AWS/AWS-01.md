### AWS Workshop
#### AWS - 1 - 基础知识和服务介绍
云计算 -- 按需提供资源（计算资源、存储资源、IT资源），也即是云服务

基础概念：
	Region  
		不同的地理区域，至少包含两个以上的可用区（AZ）
	AZ<Available Zone>
		一个或多个数据机房
		eg: us-east-1a -- us-east-1 + a
	ARN<Amazon Resource Name>
		AWS资源唯一标识
	
AWS服务限制：
服务数量
Region区别
使用TrustAdvisor检查限制情况
可对软性限制提出调整

使用
[CloudPing]: https://www.cloudping.info/

检测不同region对当前区域的延迟



使用AWSCLI在命令行操作AWS



AWS服务与解决方案：
	计算
		EC2
		Lambda
		ELB
		Auto Scaling
	存储
		S3
	数据库
		RDS<关系型数据库>
		DynamoDB<NoSQL数据库>
		ElastiCache<内存缓存>
	网络和内容分发
		CloudFront<CDN>
	开发人员工具
		CodeDeploy<自动代码部署>
	管理工具
		CloudWatch<资源和应用程序监控>
		CloudFormation<AWS资源创建模板>
	安全性、身法与合规性
		IAM<AWS访问控制>
	分析
		Elastic MapReduce<托管的 Hadoop 框架>
	应用程序服务
		API Gateway
	消息收发
		SQS<消息队列>
	物联网
	支持<技术支持等>
	机器学习
	
XaaS:
	IaaS: infrastructure as a service<AWS>
	PaaS: platform as a service <AWS Elastic Beanstalk>
	SaaS: sofeware as a service <web端的google docs/xmind/office365>
	FaaS: function as a service <AWS Lambda>

#### AWS - 2 - IAM基础

控制AWS资源的权限。
基础概念
	用户
	用户组 - 对用户权限的控制(比如创建EC2)
	角色 - 对资源权限的控制(比如限定EC2能不能访问S3等)
	策略
		托管策略/managed policies
		内联策略/inline plicies
	策略可以直接挂到用户、用户组、角色上
	
root用户<账户所有者> -- 拥有所有资源的的权限
IAM用户<通过IAM创建的> -- 通过设置策略来限制用户的权限

一般不直接使用root用户，而是使用IAM用户

敏感用户设置MFA(google two factor authentication)

#### AWS - 3 - 部署静态网站到EC2

1. 使用hexo生成个人静态网站
2. 创建EC2
   1. 选择AMI(系统镜像 可自己生成保存供下次使用)
   2. 选择Type - 使用目的 GPU 内存等
   3. 配置网络(VPC)  ROLE 
   4. 添加额外Storage
   5. Add TAGS
   6. CONFIG SECURITY GROUP
   7. REVIEW AND LAUNCH
3. 使用EC2生成的公钥访问EC2 - SSH
4. 按照Nginx && scp本地静态网站到服务器
5. 使用公网IP或者域名访问



