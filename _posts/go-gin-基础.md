---
title: go-gin-基础
date: 2020-1-14 14:05:44
tags: go gin
categories: go gin
---

# 获取`GET` `path`参数
``` go
  func main() {
    r := gin.Default()
    r.GET("/index/:id", func(context *gin.Context){
      r.JOSN(http.StatusOK, gin.H{
        "id": context.Param("id"),
      })
    })
  }
```

# 获取`POST` `raw json`参数

1. 定义一个公用`struct`，用于标准返回数据格式
``` go
  type ResBodyS struct {
    ErrCode int `json:"errCode"`
    Desc string `json:"description" binding:"required"` //参数存在性校验
  }
```
2. 定义针对于特定接口的`struct`
``` go
  type IndexReqBodyS struct {
    ResBodyS
    Data string `json:"index"`
  }
```
3. 使用`context.BindJSON()`解析请求中的body参数
``` go
  func main() {
    r := gin,Default()
    r.POST("/index", func(context *gin.Context){
      var reqInfo IndexReqBodyS
      err := context.BindJSON(&reqInfo)
      if err != nil {
        fmt.PrintFln("parse json data error! err:", err)
        return
      }
      context.JSON(http.StatusOK, reqInfo)
    })
    r.Run(":9090")
  }
```

# 捕获异常
在 Go 中异常就是`panic`，它是在程序运行的时候抛出的，当`panic`抛出之后，如果在程序里没有添加任何保护措施的话，控制台就会在打印出`panic`的详细情况，然后终止运行。
当程序发生`panic`后，在`defer`(延迟函数) 内部可以调用`recover`进行捕获。
``` go
  defer func() {
    err := recover();
    if err != nil {
      fmp.Println(err)
    }
  }
```

# 单元测试
使用`net/http/httptest`包
与原生测试相比，使用`gin.Engine.ServeHTTP`发送请求(原生使用`http.HandlerFunc.ServeHTTP`)

``` go
var engine *gin.Engine

const accessToken = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1ODA5OTE1NTUsInVzZXJfaWQiOjE4LCJuYW1lIjoidGVzdDEifQ.900X9t3qveW0NGn3_FhYxHXlgsLAdl1BkjfKxNlBNyE"

func CreateRequest(method constant.HttpRequestMethod, url string, params interface{}) (status constant.ResultCode, resBody *util.ResS, returnError error) {
	if engine == nil {
		gin.SetMode(gin.ReleaseMode)
		engine = gin.New()
		database.SetUpDatabase()
		controller.SetupRouter(engine)
	}

	var (
		rr        = httptest.NewRecorder()
		req       = new(http.Request)
		err error = nil
	)
	if params != nil {
		tmp, err := json.Encode(params)
		if err != nil {
			return constant.FAILED, nil, err
		}
		req, err = http.NewRequest(string(method), url, bytes.NewBuffer([]byte(tmp)))
	} else {
		req, err = http.NewRequest(string(method), url, nil)
	}
	if err != nil {
		return constant.FAILED, nil, err
	}

	req.Header.Add("Authorization", fmt.Sprintf("Bear %s", accessToken))
	engine.ServeHTTP(rr, req)

	formatRes := util.ResS{}
	err = json.Decode(rr.Body.Bytes(), &formatRes)
	if err != nil {
		return constant.FAILED, nil, err
	}
	return constant.SUCCESS, &formatRes, nil
}

func CreateTest(t *testing.T, method constant.HttpRequestMethod, url string, params interface{}) {
	_, resBody, err := CreateRequest(method, url, params)
	if err != nil {
		t.Errorf("got error: %s", err.Error())
		return
	}
	if resBody != nil {
		if resBody.Code != constant.SUCCESS {
			t.Errorf("GetInfo test failed, got: %v, info: %s", resBody.Data, resBody.Message)
		}
	}
}
```