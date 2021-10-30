# Golang  编程思维和工程实践


## 1.优雅的实现构造函数的编程思想

参考：
https://commandcenter.blogspot.com/2014/01/self-referential-functions-and-design.html

通过这个方式可以方便构造不同对象，同时避免了大量重复代码

```go
package main

import (
	"fmt"
	"time"

	"golang.org/x/net/context"
)

type ClusterNew struct {
	opts options
}
type options struct {
	connectionTimeout time.Duration
	readTimeout       time.Duration
	writeTimeout      time.Duration
	logError          func(ctx context.Context, err error)
}

//通过一个选项实现为一个函数指针来达到一个目的：设置选项中的数据状态
type Option func(c *options)

//设置某个参数的一个具体实现  用到了闭包的用法
func LogError(f func(ctx context.Context, err error)) Option {
	return func(opts *options) {
		opts.logError = f
	}
}

//对关键数据变量的赋值采用一个方法实现而不是直接设置
func ConnectionTimeout(d time.Duration) Option {
	return func(opts *options) {
		opts.connectionTimeout = d
	}
}

func WriteTimeout(d time.Duration) Option {
	return func(opts *options) {
		opts.writeTimeout = d
	}
}

func ReadTimeout(d time.Duration) Option {
	return func(opts *options) {
		opts.readTimeout = d
	}
}

//构造函数具体实现 传入相关 Options new 一个对象赋值
//如果参数很多 也不需要传入很多参数 只需要传入 opts...Option 即可
func NewCluster(opts ...Option) *ClusterNew {
	clusterOpts := options{}
	for _, opt := range opts {
		//函数指针的赋值调用
		opt(&clusterOpts)
	}

	cluster_obj := new(ClusterNew)
	cluster_obj.opts = clusterOpts
	return cluster_obj
}
func main() {

	//前期储备 设定相关参数
	commonsOpts := []Option{
		ConnectionTimeout(1 * time.Second),
		ReadTimeout(2 * time.Second),
		WriteTimeout(3 * time.Second),
		LogError(func(ctx context.Context, err error) {}),
	}

	//终极操作 构造函数
	cluster_obj := NewCluster(commonsOpts...)

	//测试验证
	fmt.Println(cluster_obj.opts.connectionTimeout)
	fmt.Println(cluster_obj.opts.writeTimeout)

	//输出
	//1s
	//3s
}


```



## 2.优雅的实现继承编程思想

Golang 里面没有 C++、Java 那种继承的实现方式，但是，我们可以通过 Golang 的匿名组合来实现继承，这里需要注意
，这个是实际编程中经常用到的一种姿势。

具体实现就是一个 struct 里面包含一个匿名的 struct 也就是通过匿名组合 这最基础的基类就是一个 struct 结构

然后定义相关成员变量，然后在定义一个子类，也是一个 struct 里面包含了前面的 struct 即可实现继承。

实例代码如下
```go

package main

import "fmt"

//定义一个基类 struct 的 MsgBase 里面包含一个成员变量 msgID

type MsgBase struct {
	msgId   int
	msgType int
}

//MsgBase 的一个成员方法  用来设置 msgID
func (msg *MsgBase) SetId(msgId_ int) {
	msg.msgId = msgId_
}

func (msg *MsgBase) SetType(msgType_ int) {
	msg.msgType = msgType_
}

//子类 再定义一个 struct msgSub 包含了 MsgBase 但是并没有给定 MsgBase 任何名字 因此是匿名组合
type MsgSub struct {
	MsgBase

	//如果子类也包含了一个基类的一样的成员变量，那么通过子类设置和获取得到的变量都是基类的成员
	msgId int
}

func (msb *MsgSub) GetId() int {
	return msb.msgId
}

func main() {
	msb := &MsgSub{}

	msb.SetId(123)
	msb.SetType(1)

	fmt.Println("msb.msgID =", msb.msgId, "\t msb.MsgBase.msgId=", msb.MsgBase.msgId)
	fmt.Println("msb.msgType =", msb.msgType, "\t msb.MsgBase.msgType=", msb.MsgBase.msgType)

	//输出
	//msb.msgID = 0    msb.MsgBase.msgId= 123
	//msb.msgType = 1          msb.MsgBase.msgType= 1
}
```
