

# Go modules

普大喜奔的是，从 Go 1.11 版本开始，官方已内置了更为强大的 [Go modules](https://golang.org/cmd/go/#hdr-Modules__module_versions__and_more) 来一统多年来 Go 包依赖管理混乱的局面(Go 官方之前推出的 [dep](https://github.com/golang/dep) 工具也几乎胎死腹中)，并且将在 1.13 版本中正式默认开启。

目前已受到社区的看好和强烈推荐，建议新项目采用 Go modules。



## go mod命令

| command  | usage                                              |
| -------- | -------------------------------------------------- |
| download | 下载依赖的module到本地cache                        |
| edit     | 编辑go.mod                                         |
| graph    | 打印模块依赖图                                     |
| init     | 在当前文件夹下初始化一个新的module，创建go.mod文件 |
| vendor   | 将依赖复制到vendor下                               |
| verify   | 依赖校验                                           |
| why      | 解释为什么需要依赖                                 |
| tidy     | 增加丢失的module，去掉未使用的module               |





## govendor子命令

| 子命令  | 功能                                                         |
| ------- | ------------------------------------------------------------ |
| init    | 创建 `vendor` 目录和 `vendor.json` 文件                      |
| list    | 列出&过滤依赖包及其状态                                      |
| add     | 从 `$GOPATH` 复制包到项目 `vendor` 目录                      |
| update  | 从 `$GOPATH` 更新依赖包到项目 `vendor` 目录                  |
| remove  | 从 `vendor` 目录移除依赖的包                                 |
| status  | 列出所有缺失、过期和修改过的包                               |
| fetch   | 从远程仓库添加或更新包到项目 `vendor` 目录(不会存储到 `$GOPATH`) |
| sync    | 根据 `vendor.json` 拉取相匹配的包到 `vendor` 目录            |
| migrate | 从其他基于 `vendor` 实现的包管理工具中一键迁移               |
| get     | 与 `go get` 类似，将包下载到 `$GOPATH`，再将依赖包复制到 `vendor` 目录 |
| license | 列出所有依赖包的 LICENSE                                     |
| shell   | 可一次性运行多个 `govendor` 命令                             |

## govendor状态参数

| 状态      | 缩写 | 含义                                                 |
| --------- | ---- | ---------------------------------------------------- |
| +local    | l    | 本地包，即项目内部编写的包                           |
| +external | e    | 外部包，即在 `GOPATH` 中、却不在项目 `vendor` 目录   |
| +vendor   | v    | 已在 `vendor` 目录下的包                             |
| +std      | s    | 标准库里的包                                         |
| +excluded | x    | 明确被排除的外部包                                   |
| +unused   | u    | 未使用的包，即在 `vendor` 目录下，但项目中并未引用到 |
| +missing  | m    | 被引用了但却找不到的包                               |
| +program  | p    | 主程序包，即可被编译为执行文件的包                   |
| +outside  |      | 相当于状态为 `+external +missing`                    |
| +all      |      | 所有包                                               |