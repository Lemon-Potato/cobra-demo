[原文](https://github.com/spf13/cobra-cli/blob/main/README.md)

## Cobra 生成器
- Cobra 本身提供程序用来创建项目和新增命令。可以轻易地将 Cobra 纳入你的项目。
- 可以通过 `go install github.com/spf13/cobra-cli@latest` 命令来安装 Cobra 生成器。Golang 会自动将该命令加入 `$GOPATH/bin` 目录下
- 在命令行工具下输入 `cobra-cli` 可看到该工具
- Cobra 生成器目前只支持两种命令

#### cobra-cli init
- `cobra-cli init [app]` 命令会为你创建代码，方便高效地以正确的格式将 Cobra 引入你的项目。同时它也可以引入指定 License 加入你的项目
- 通过 Go modules，可以方便地结合 Cobra 生成器。生成器基于 Go modules 工作

##### 使用步骤
- 初始化 module
    ```
    cd $HOME/code 
    mkdir myapp
    cd myapp
    go mod init github.com/spf13/myapp
    ```
- 通过 Cobra CLI 初始化 
    ```
    cd $HOME/code/myapp
    cobra-cli init
    go run main.go
    ```
  - cobra-cli init 的一些参数
    - `--author` 可以声明作者
      - `cobra-cli init --author "Steve Francia spf@spf13.com"`
    - `--license` 可以声明 license
      - `cobra-cli init --license apache`
    - `--viper` 自动初始化引入 [viper](https://github.com/spf13/viper)


- 通过 Cobra CLI 新增命令
    ```
    cobra-cli add serve
    cobra-cli add config
    cobra-cli add create -p 'configCmd'
    ```
    - `cobra-cli add` 支持 `cobra-cli init` 参数
    - `-p` 指定父命令，相当于在父命令下的子命令。注意格式为 *Cmd 不能忘记 Cmd
      - `cobra-cli add add-user` 是错的。`cobra-cli add addUser` 才是对的


- 通过 go run main.go 运行

##### 配置文件 .cobra.yaml
- Cobra 生成器可以使用配置文件，方便一些基本配置
- 文件内容如下

```
author: Steve Francia <spf@spf13.com>
license: MIT
useViper: true
```
