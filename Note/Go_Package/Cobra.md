## 介绍

命令行界面 (CLI) 应用程序是在终端或命令提示符上运行的软件程序。用户通过指定命令并接收文本作为反馈来与软件交互。Cobra就是用于构建属于自己的命令行。
## 使用Cobra软件包
### 安装Cobra软件包
`{go icon title:install_cobra} go install github.com/spf13/cobra-cli@latest`
### 初始化项目
`{go icon title:init_cobra}cobra-cli init`
### 添加指令
`{go icon title:add_cmd}cobra-cli add <cmd_name>`
## 命令修改

>  主要修改Use，Short，Long，Args，Run

```go title:replace_settings.go 
var timezoneCmd = &cobra.Command{
	Use:   "timezone",
	Short: "Get the current time in a given timezone",
	Long: `Get the current time in a given timezone.

This command takes one argument, the timezone you want to get the current time in.
It returns the current time in RFC1123 format.`,
	Args: cobra.ExactArgs(1), // 正好 1 个参数
	Run: func(cmd *cobra.Command, args []string) {
		timezone := args[0]
		currentTIme, err := getTimeInTimeZone(timezone)
		if err != nil {
			log.Fatalln("The timezone string is invalid")
		}
		fmt.Println(currentTIme)
	},
}
```
### 使用
`{go icon title:use_cobra} go build -o <name.exe> | .\<name.exe> <name_cmd> <args>`
### 添加Flags
#### 持久Flag

* PersistentFlag是自己命令和子命令都可以使用的Flag
`{go icon title:persistentFlag}rootCmd.PersistentFlags.String("name", "vale", "descriptione") `
#### 绑定使用
* 在 init 区需要进行绑定
`{go icon title:bindFlags}viper.BindPFlag("name", rootCmd.Flags.Lookup("name"))`

**除了PersistentFlag，还有Flag(本地标志)，子命令无法获取**
#### 设置必须标识

```go title:require_flag.go 
rootCmd.Flags().StringVarP(&Region, "region", "r", "", "AWS region (required)")
rootCmd.MarkFlagRequired("region") 
```

### 参数自定义

- `NoArgs` - 如果存在任何位置参数，该命令将报告错误。
- `ArbitraryArgs` - 该命令将接受任何 args。
- ` OnlyValidArgs` - 如果存在任何位置参数不在 `Command` 的 `ValidArgs` 字段中，则命令将报告错误。
- `MinimumNArgs（int）` - 如果没有至少 N 个位置参数，该命令将报告错误。
- `MaximumNArgs（int）` - 如果位置参数超过 N 个，该命令将报告错误。
- `ExactArgs（int）` - 如果位置参数不完全是 N 个，该命令将报告错误。
- `ExactValidArgs（int）` = 如果 Command 的 `ValidArgs` 字段中没有恰好 N 个位置参数，或者有任何位置参数不在 `Command` 的 ValidArgs 字段中，则命令将报告并出错
- `RangeArgs（min， max）` - 如果 args 的数量不在预期 args 的最小和最大数量之间，该命令将报告错误。