# go

1. - 相关资源放置在 /opt/ 目录下,
   - `wget https://dl.google.com/go/go1.11.4.linux-amd64.tar.gz`
   - 加压后配置 go 的工作空间和环境变量
   - `mkdir /opt/gocode/{src,bin,pkg}`
   - `vi /etc/profile`
   ```yaml
   export GOROOT=/opt/go           #Golang源代码目录，安装目录
   export GOPATH=/opt/gocode       #Golang项目代码目录
   export PATH=$GOROOT/bin:$PATH   #Linux环境变量
   export GOBIN=$GOPATH/bin        #go install后生成的可执行命令存放路径
   ```
   - `source /etc/profile`

2. - `cd /opt/gocdode/src/`
   - `mkdir testgo && cd testgo`
   -  `vi main.go`
   ```
   package main
   import "fmt"
   func main(){
       fmt.Println("dsdfgiuydghfjgkh")
   }
   ```
   
   - `go run main.go`
   - `go build main.go`
   - `./main.go`
