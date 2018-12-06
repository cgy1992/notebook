# 配置调试项目

如果当前目录下有多个 flutter 工程，vscode 默认的是第一个创建的。这个时候可以通过配置调试选项来选择调试不同的项目。

![](http://blogqn.maintel.cn/TIM截图20181205121055.png?e=3120783080&token=cs2nCfx72Y7hW0_NpFYzb3Jab90IJWraRtphMd-q:YGXO97fcNhjK5LiFZfQx8R-G7vE=)

选择 `add Configuration`，在 `"configurations"` 中添加：

```
        {
            "name": "test1",
            "program": "test_1/lib/main.dart",
            "request": "launch",
            "type": "dart"
        }
```

- name 表示工程名
- program 工程入口，即main文件所在路径
- request 可选 launch 或 attach，目前的项目只支持 launch
- type 配置类型