# JWT 接入

### 背景

###### 从单体应用架构到分布式应用架构再到微服务架构，应用的安全访问在不断的经受考验。为了适应架构的变化、需求的变化，身份认证与鉴权方案也在不断的变革。面对数十个甚至上百个微服务之间的调用，如何保证高效安全的身份认证？面对外部的服务访问，该如何提供细粒度的鉴权方案？本文将会为大家阐述微服务架构下的安全认证与鉴权方案。

### 目标

###### 达到接口鉴权，保证系统的安全性

### 实践

1. 创建验证表 和 表内数据：

```mysql
CREATE TABLE `blog_auth` (
  `id` int unsigned NOT NULL AUTO_INCREMENT,
  `app_key` varchar(20) DEFAULT '' COMMENT 'Key',
  `app_secret` varchar(50) DEFAULT '' COMMENT 'Secret',
  `sub` varchar(50) DEFAULT '' COMMENT 'Sub',
  `name` varchar(50) DEFAULT '' COMMENT 'Name',
  `is_del` tinyint DEFAULT '0' COMMENT '是否删除',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 2 DEFAULT CHARSET = utf8mb4 COLLATE = utf8mb4_0900_ai_ci COMMENT = '认证管理';


insert into `blog_auth` values(1,'logan','loganMasecret','gindemo','LoganMa');
```

2. 安装JWT并在 `/internal/model`下创建实体类

   ```go
   go get -u github.com/dgrijalva/jwt-go@v3.2.0 
   
   type Auth struct {
   	*Model
   	AppKey    string `json:"app_key"`
   	AppSecret string `json:"app_secret"`
     Name      string `json:"name"`
   	Sub       string `json:"sub"`
   }
   
   func (Auth) TableName() string {
   	return "blog_auth"
   }
   ```

3. **其核心步骤是根据`app_key`和`app_secret`进行校验并生成`token`，并在其他请求时携带该参数，并做到<u>鉴权</u>功能。**

   ```go
   // 请求入口
   func GetAuth(c *gin.Context) {
   	params := service.AuthRequest{}
   	response := app.NewResponse(c)
   	valid, errs := app.BindAndValid(c, &params) // 校验参数
   	if valid {
   		global.Logger.Errorf("app.BindAndValid errs: %+v", errs)
   		response.ToErrorResponse(errcode.InvalidParams)
   		return
   	}
   	svc := service.New(c.Request.Context())
   	err := svc.CheckAuth(&params) // 检查Auth
   	if err != nil {
   		global.Logger.Errorf("svc.CheckAuth err: %+v", err)
   		response.ToErrorResponse(errcode.UnauthorizedAuthNotExit)
   		return
   	}
   	token, err := app.GenerateToken(params.AppKey, params.AppSecret) //生成jwt
   	if err != nil {
   		global.Logger.Errorf("app.GenerateToken err: %+v", err)
   		response.ToErrorResponse(errcode.UnauthorizedTokenGenerate)
   		return
   	}
   
   	response.ToResponse(gin.H{
   		"token": token, // 返回token
   	})
   }
   ```

4. 生成`token`方法解释：对`app_key`和`app_secret`加密并对`token`设置过期时间，最后进行签名认证

   ```go
   func GenerateToken(appKey, appSecret string) (string, error) {
   	nowTime := time.Now()
   	expireTime := nowTime.Add(global.JWTSetting.Expire)
   	claims := Claims{
   		AppKey:    util.EncodeMD5(appKey),
   		AppSecret: util.EncodeMD5(appSecret),
   		StandardClaims: jwt.StandardClaims{
   			ExpiresAt: expireTime.Unix(),
   			Issuer:    global.JWTSetting.Issuer,
   		},
   	}
   
   	tokenClaims := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
   	token, err := tokenClaims.SignedString(GetJWTSecret()) // 签名
   	return token, err
   }
   ```

5. 请求接口时，使用`gin`中间件的模式进行`token`校验，达到鉴权的目的。

   ```go
   
   func JWT() gin.HandlerFunc {
   	return func(c *gin.Context) {
   		var (
   			token string
   			ecode = errcode.Success
   		)
   		if s, exist := c.GetQuery("token"); exist { // 从Request Param获取
   			token = s
   		} else {
   			token = c.GetHeader("token") // 没拿到，再从Header中获取
   		}
   		if token == "" {
   			ecode = errcode.NotAuth
   		} else {
   			_, err := app.ParseToken(token)
   			if err != nil {
   				switch err.(*jwt.ValidationError).Errors {
   				case jwt.ValidationErrorExpired: // 
   					ecode = errcode.UnauthorizedTokenTimeout
   				default:
   					ecode = errcode.UnauthorizedTokenError
   				}
   			}
   		}
   		if ecode != errcode.Success {
   			response := app.NewResponse(c)
   			if ecode == errcode.NotAuth {
   				response.ToErrorResponse(ecode.WithDetails("please visit /api/v1/auth "))
   			} else {
   				response.ToErrorResponse(ecode)
   			}
   			c.Abort()
   			return
   		}
   		c.Next()
   	}
   }
   
   // 解析token, 并验证是否可用
   func ParseToken(token string) (*Claims, error) {
   	tokenClaims, err := jwt.ParseWithClaims(token, &Claims{}, func(token *jwt.Token) (interface{}, error) {
   		return GetJWTSecret(), nil
   	})
   	if tokenClaims != nil {
   		if claims, ok := tokenClaims.Claims.(*Claims); ok && tokenClaims.Valid {
   			return claims, nil
   		}
   	}
   	return nil, err
   }
   ```

   Tips：实际结果可以参考`github`进行下载`gindemo` https://github.com/maronghe/gindemo



### 总结

改善 & TODO list

- [ ] 当前版本发现日志打印的格式不清晰，如【报错行数不显示】【时间格式及时间重复等问题】
- [ ] 