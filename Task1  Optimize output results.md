以下内容为”优化输出“这一任务的实现思路

文字在显示时可以分为三部分：`字形码，前景色、背景色`；

在go语言中，通过在字符串前加上`\x1b[arg1;arg2;arg3m`，在终端中其后的所有字符都会被设置为相应格式，其中可选参数arg1、arg2、arg3分别用于设置通用格式、前景色、背景色，对应的意义见下图：

<img src="https://user-images.githubusercontent.com/52828870/125442080-057ca057-d66a-441a-ac23-7a904431083f.png" alt="image-20210713182547452" style="zoom:50%;" />
所以使用`fmt.Println("\x1b[0,32mHello world!")`可以输出绿色的<font color='green'>Hello world!</font> 

因此可以使用这一方式来完成”优化输出“这一任务，起初我打算直接设置命令行参数为相应的格式，例如：`Name:"\x1b[0,32m config \x1b[0m"`，虽然这样可以输出预期的效果，但此时控制参数也成了命令的一部分，使得参数config无法被正常使用。

幸运的是，urfave/cli是通过默认的模板来设置提示文档的格式的，通过设置cli中的AppHelpTemplate、CommandHelpTemplate、SubcommandHelpTemplate字段可以改变提示文档的样式，在[社区文档](https://pkg.go.dev/text/template)中提供了template的语法，在cli默认的AppHelpTemplate变量中对于COMMANDS的template如下：

```go
`{{if .VisibleCommands}}

COMMANDS:{{range .VisibleCategories}}{{if .Name}}

   {{.Name}}:{{range .VisibleCommands}}
     {{join .Names ", "}}{{"\t"}}{{.Usage}}{{end}}{{else}}{{range .VisibleCommands}}
   {{join .Names ", "}}{{"\t"}}{{.Usage}}{{end}}{{end}}{{end}}{{end}}{{if .VisibleFlags}}
`
```

为了设置输出样式，这个默认模板的代码显得很乱，将代码格式化后如下：

```go
`{{if .VisibleCommands}}

	COMMANDS:
	{{range .VisibleCategories}}
		{{if .Name}}
   			{{.Name}}:
   			{{range .VisibleCommands}}
     			{{join .Names ", "}}{{"\t"}}{{.Usage}}{{end}}
     	{{else}}
     		{{range .VisibleCommands}}
   				{{join .Names ", "}}{{"\t"}}{{.Usage}}{{end}}
   		{{end}}
   	{{end}}
{{end}}`
```

这样代码逻辑就清晰多了，在此基础上添加格式控制字符，我先设置了所需的各个颜色变量：

```go
const TextFormat = `
{{- $bold	:="\x1b[1m" -}}
{{- $black	:="\x1b[0;30m" -}}	{{- $Black	:="\x1b[1;30m" -}}	
{{- $red	:="\x1b[0;31m" -}}	{{- $Red	:="\x1b[1;31m" -}}
{{- $green	:="\x1b[0;32m" -}}	{{- $Green	:="\x1b[1;32m" -}}
{{- $yellow	:="\x1b[0;33m" -}}	{{- $Yellow	:="\x1b[1;33m" -}}
{{- $blue	:="\x1b[0;34m" -}}	{{- $Blue	:="\x1b[1;34m" -}}
{{- $pruple	:="\x1b[0;35m" -}}	{{- $Pruple	:="\x1b[1;35m" -}}
{{- $cyan	:="\x1b[0;36m" -}}	{{- $Cyan	:="\x1b[1;36m" -}}
{{- $white	:="\x1b[0;37m" -}}	{{- $White	:="\x1b[1;37m" -}}
{{- $plain	:="\x1b[0m" -}}
`
```

当需要设置某一个文本的格式时，便只需在前面添加相应的变量名即可。例如`{{$Green}}COMMANDS:{{$plain}}`可以设置`COMMANDS:`为粗体绿色。通过这样的方式能很方便的实现"优化输出"这一要求。
例：`go run main.go help`

![image](https://user-images.githubusercontent.com/52828870/125442343-dc8b26f8-8e3d-4add-a539-a59fb7726440.png)

例：`go run main.go endpoint`

![image](https://user-images.githubusercontent.com/52828870/125442418-3dcc3a74-c12c-474e-b15a-e22a3b211602.png)

例：`go run main.go endpoint ls -h`

![image](https://user-images.githubusercontent.com/52828870/125442467-490d0e11-2f60-4909-a58d-f16c22ff876d.png)


