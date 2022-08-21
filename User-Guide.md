 [原文]{https://github.com/spf13/cobra/blob/main/user_guide.md}

 ## 使用引导
 - 基本文件结构
    ```
    ▾ appName/
        ▾ cmd/
            add.go
            your.go
            commands.go
            here.go
            main.go
    ```
- 初始化 Cobra
    ```
    package main

    import (
    "{pathToYourApp}/cmd"
    )

    func main() {
    cmd.Execute() // 初始化 Cobra 整个项目
    }
    ```

#### Cobra 基本使用
##### 创建额外命令
- 通过 `cobra-cli add [命令]` 可初始化对应命令
##### 返回与处理异常
- `RunE` 可以用来处理异常，方法本质跟 `Run` 类似，增加了 error 参数 

##### 处理参数 Flags
- 参数可用来控制对应命令的操作
- 为了赋值 flag，我们需要定义变量
    ```
    var Verbose bool
    var Source string
    ```

###### 持久化参数 (Persistent Flags)
- 持久化的参数可用于所有的命令，需要在 root.go 当中声明
    ```
    rootCmd.PersistentFlags().BoolVarP(&Verbose, "verbose", "v", false, "verbose output")
    ```
###### 本地参数 (Local Flags)
- 该参数只适用于该命令
    ```
    localCmd.Flags().StringVarP(&Source, "source", "s", "", "Source directory to read from")
    ```
- 严格意义上，local Flags 只能本命令使用，但是在启动 `Command.TraverseChildren` 配置时，Cobra 会解析 local Flags 到各个命令
    ```
    command := cobra.Command{
    Use: "print [OPTIONS] [COMMANDS]",
    TraverseChildren: true,
    }
    ```
###### 通过 viper 设置参数
```
var author string

func init() {
  rootCmd.PersistentFlags().StringVar(&author, "author", "YOUR NAME", "Author name for copyright attribution")
  viper.BindPFlag("author", rootCmd.PersistentFlags().Lookup("author"))
}
```

###### 限制参数必须
```
// local Flags 本地参数
rootCmd.Flags().StringVarP(&Region, "region", "r", "", "AWS region (required)")
rootCmd.MarkFlagRequired("region")
```

```
// persistent Flags 持久化参数
rootCmd.PersistentFlags().StringVarP(&Region, "region", "r", "", "AWS region (required)")
rootCmd.MarkPersistentFlagRequired("region")
```

###### 限制参数必须同时有
- 例如在使用 username 同时需要 password。我们可以限制它们必须同时出现
- 
```
rootCmd.Flags().StringVarP(&u, "username", "u", "", "Username (required if password is set)")
rootCmd.Flags().StringVarP(&pw, "password", "p", "", "Password (required if username is set)")
rootCmd.MarkFlagsRequiredTogether("username", "password") // 关键
```
- 当然，多选一的情况也是可能存在的，例如配置文件只读一个
```
rootCmd.Flags().BoolVar(&u, "json", false, "Output in JSON")
rootCmd.Flags().BoolVar(&pw, "yaml", false, "Output in YAML")
rootCmd.MarkFlagsMutuallyExclusive("json", "yaml")
```

###### 位置参数和自定义参数
- NoArgs-如果存在任何位置参数，该命令将报告错误。
- ArbitraryArgs-该命令将接受任何args。
- OnlyValidArgs-如果有任何位置参数不在命令的ValidArgs字段中，则该命令将报告错误。
- MinimumNArgs（int）-如果没有至少N个位置参数，则该命令将报告错误。
- MaximumNArgs（int）-如果位置参数超过N个，则该命令将报告错误。
- ExactArgs（int）-如果没有正好N个位置参数，则命令将报告错误。
- ExactValidArgs（int）-如果没有正好N个位置参数，或者如果有任何位置参数不在命令的ValidArgs字段中，则该命令将报告错误
- RangeArgs（min，max）-如果args的数目不在预期的最小和最大args数目之间，则命令将报告错误。

```
var cmd = &cobra.Command{
  Short: "hello",
  Args: func(cmd *cobra.Command, args []string) error {
    if len(args) < 1 {
      return errors.New("requires a color argument")
    }
    if myapp.IsValidColor(args[0]) {
      return nil
    }
    return fmt.Errorf("invalid color specified: %s", args[0])
  },
  Run: func(cmd *cobra.Command, args []string) {
    fmt.Println("Hello, World!")
  },
}
```

#### 钩子
- 顾名思义，用在不同阶段触发的钩子
```
PersistentPreRun
PreRun
Run
PostRun
PersistentPostRun
```

```
package main

import (
  "fmt"

  "github.com/spf13/cobra"
)

func main() {

  var rootCmd = &cobra.Command{
    Use:   "root [sub]",
    Short: "My root command",
    PersistentPreRun: func(cmd *cobra.Command, args []string) {
      fmt.Printf("Inside rootCmd PersistentPreRun with args: %v\n", args)
    },
    PreRun: func(cmd *cobra.Command, args []string) {
      fmt.Printf("Inside rootCmd PreRun with args: %v\n", args)
    },
    Run: func(cmd *cobra.Command, args []string) {
      fmt.Printf("Inside rootCmd Run with args: %v\n", args)
    },
    PostRun: func(cmd *cobra.Command, args []string) {
      fmt.Printf("Inside rootCmd PostRun with args: %v\n", args)
    },
    PersistentPostRun: func(cmd *cobra.Command, args []string) {
      fmt.Printf("Inside rootCmd PersistentPostRun with args: %v\n", args)
    },
  }

  var subCmd = &cobra.Command{
    Use:   "sub [no options!]",
    Short: "My subcommand",
    PreRun: func(cmd *cobra.Command, args []string) {
      fmt.Printf("Inside subCmd PreRun with args: %v\n", args)
    },
    Run: func(cmd *cobra.Command, args []string) {
      fmt.Printf("Inside subCmd Run with args: %v\n", args)
    },
    PostRun: func(cmd *cobra.Command, args []string) {
      fmt.Printf("Inside subCmd PostRun with args: %v\n", args)
    },
    PersistentPostRun: func(cmd *cobra.Command, args []string) {
      fmt.Printf("Inside subCmd PersistentPostRun with args: %v\n", args)
    },
  }

  rootCmd.AddCommand(subCmd)

  rootCmd.SetArgs([]string{""})
  rootCmd.Execute()
  fmt.Println()
  rootCmd.SetArgs([]string{"sub", "arg1", "arg2"})
  rootCmd.Execute()
}
```