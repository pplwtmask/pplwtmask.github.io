---
title: AWS Lambda
date: 2023-06-23 15:38:06
---

> 最近要用Lambda部署一个node项目，折腾了好久。。。简单记录一下

# 部署工具

可以使用AWS官方提供的工具部署，但是考虑到其它平台的兼容性使用了[Serverless Framework](https://github.com/serverless/serverless)


# 调用

针对非HTTP API：
使用了AWS Cognito的鉴权服务，然后使用的AWS Amplify API调用，配置
```
export default {
	cognito: {
		REGION: 'us-east-1',
		USER_POOL_ID: 'us-east-1_xxx',
		APP_CLIENT_ID: 'xxxxxxxx',
		IDENTITY_POOL_ID: 'us-east-1:xxxxxxxx'
	},
	endpoints: [			
		{
			name: 'xxxxxxx',
			endpoint: "xxxxxxxxx", // 这里有点奇怪折腾了好久，可以参考官网文档 https://docs.aws.amazon.com/lambda/latest/dg/API_Invoke.html
			service: "lambda",
			region: "us-east-1"
		}
	],
	oauth:{
		// Domain name
		domain: 'xxxxxxxxxxx',
	
		// Authorized scopes
		scope: ['phone', 'email', 'profile', 'openid', 'aws.cognito.signin.user.admin'],
	
		// Callback URL
		redirectSignIn: 'xxxxxxxxx', // or 'exp://127.0.0.1:19000/--/', 'myapp://main/'
	
		// Sign out URL
		redirectSignOut: 'xxxxxxxxxxxxxxxxxxxxx', // or 'exp://127.0.0.1:19000/--/', 'myapp://main/'
	
		// 'code' for Authorization code grant, 
		// 'token' for Implicit grant
		// Note that REFRESH token will only be generated when the responseType is code
		responseType: 'token',
	
		// optional, for Cognito hosted ui specified options
		options: {
			// Indicates if the data collection is enabled to support Cognito advanced security features. By default, this flag is set to true.
			AdvancedSecurityDataCollectionFlag: true
		},
	
		urlOpener: null
	}

};

```











