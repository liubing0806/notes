刷新token主要分为两种情况:
- 有回调地址
	有回调地址也分为两种情况
	- 需要部署服务到云内
		单独开发对应的中转服务,在请求时额外添加参数来区分,后续接口请求也是调用我们在对应平台云内的服务
	- 不需要服务部署到云内
		对于这种情况,在api平台中设置回调地址为魔方页面生成的链接(现阶段为手工拼接),要点是state的拼接,需要做到区分客户和店铺
		在客户授权完毕后,等待客户端发起token获取,后续刷新逻辑放到客户端进行
		当客户端token失效,需要重新进行授权
- 无回调地址
正常进行调用和刷新

![](https://lbnote-1304107363.cos.ap-nanjing.myqcloud.com/202307171626704.png)