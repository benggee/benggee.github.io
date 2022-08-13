## apollo使用逻辑

### 安装部署：

https://ctripcorp.github.io/apollo/#/zh/deployment/distributed-deployment-guide



### Meta Service信息：

管理工具> 系统信息

```
环境: DEV
Active: true
Meta server地址: http://121.43.234.180:8080
```



### Web界面：

http://1.1.2.1:8070 

端口可在startsh脚本里面查看



## 使用示例（Golang）

```go
import (
	"fmt"
	"github.com/philchia/agollo/v4"
)


func main() {
	apollo := agollo.NewClient(&agollo.Conf{
		AppID:          "apollo-more-appid",
		Cluster:        "default",
		NameSpaceNames: []string{"application.properties"},
		MetaAddr:       "http://1.1.2.1:8080",
	})
	apollo.Start()


	allKyes := apollo.GetAllKeys()

	fmt.Println(allKyes)


	host := apollo.GetString("mysql.dns")
	fmt.Println(host)
	user := apollo.GetString("mysql.user")
	fmt.Println(user)
	passwd := apollo.GetString("mysql.passwd")
	fmt.Println(passwd)
}
```

