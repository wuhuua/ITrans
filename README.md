#  ITrans

## :black_nib: 基于自顶向下方法可靠传输协议的实现:black_nib:

### ![brightgreen](https://img.shields.io/badge/-轻量级-brightgreen)![important](https://img.shields.io/badge/-适用于任何平台-important)![blueviolet](https://img.shields.io/badge/-rdt和GBN两种版本实现-blueviolet)![informational](https://img.shields.io/badge/-基于MuLan开源协议开源-informational)![red](https://img.shields.io/badge/-中文-red)

![Language](https://img.shields.io/badge/language-c-brightgreen)

![Documentation](https://img.shields.io/badge/documentation-yes-brightgreen)

ITrans是基于计算机自顶向下方法，原书名Computer Networking: A Top-Down Approach，第八版的第三章传输层中rdt和GBN协议的具体实现

- [实现指导书](https://media.pearsoncmg.com/aw/aw_kurose_network_3/labs/lab5/lab5.html) 


## :whale:下载须知 :feet:

```shell
git clone ${url}
```

## :articulated_lorry: 环境准备

无论在任何环境上编译，请确保你的环境拥有以下工具链：

```shell
gcc/clang gcc建议8.3以上，编译器需要支持c++20新标准
```

如果你是Mac用户，请使用homebrew更新你的bison工具，Mac原生bison工具版本落后太多

## :thinking: 如何编译:question:
```shell
gcc -Wno-return-type prog.c
```


## :alarm_clock: 如何运行
```shell
>> ./a.out

Enter the number of messages to simulate: 
Enter  packet loss probability [enter 0.0 for no loss]:
Enter packet corruption probability [0.0 for no corruption]:
Enter average time between messages from sender's layer5 [ > 0.0]:
Enter TRACE:
```
根据需要的配置填入参数即可



## :memo:实现备注

目前rdt文件夹下的实现基于rdt3.0，从A端向B端发送数据，半双工

## :sparkling_heart:License

MiniOB 采用 [木兰宽松许可证，第2版](https://license.coscl.org.cn/MulanPSL2), 可以自由拷贝和使用源码, 当做修改或分发时, 请遵守 [木兰宽松许可证，第2版](https://license.coscl.org.cn/MulanPSL2).

![License](https://img.shields.io/badge/license-MuLan-yellow)



Copyright :copyright:2023 UnloadBase

