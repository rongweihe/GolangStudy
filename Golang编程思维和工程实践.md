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
