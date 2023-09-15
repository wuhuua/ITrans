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

### rdt的实现细节
目前rdt文件夹下的实现基于rdt3.0，从A端向B端发送数据，半双工
其中rdt的实现依靠长度为一个字节的status作为状态值，定义了当前bit交换状态和等待状态，将四个状态的FSM定义为两个bit位表示的状态，因为实际上等待上层发送0号消息和等待上层发送1号消息是一致的，都作为一种状态是冗余，具体发送消息时只需要根据status当前的值就可以推断发送哪种消息
|状态 | seqNo| waiting|
| --- | --- | ---|
|等待上层发送0号消息|0|0|
|等待0号消息ACK|0|1|
|等待上层发送1号消息|1|0|
|等待1号消息ACK|1|1|
```c
/* def for a and b status
*  bytes:  -0-     -1-     -2-  -3-  -4-  -5-  -6-  -7-
*  def:   seqNo  waiting   null null null null null null
*/
```

值得注意的是在rdt3.0版本中，将不会考虑2.x与1.0版本中接收端返回与收到的包bit位不同的的ACK以表示出错，在3.0版本不接受任何与同bit位ACK不同的恢复，即在“等待0号消息ACK”状态只接受0号ACK，不接受其他任何ACK，这将导致一切问题都在超时的时候处理。
```c
if(!checkWaiting(status_a)||!checkRcv(packet)||packet.seqnum!=checkSeq(status_a)||packet.acknum==0){
return -1;
}
```

还有一点是对于原书中并列条件验证顺序的调整，比如在接收端接收数据的时候，如果bit位不正确就直接拒绝接收，不需要再遍历整个消息包验证完整性

### gbn的实现细节
gbn文件夹下的实现基于GBN的基本实现原理，半双工
gbn的实现整体严格按照书中的实现，但是在A接受ACK消息的时候，有可能因为网络波动造成ACK消息的失序，这样可能先收到大序号再收到小序号，在累积重传的机制保证下大序号一定是当前可保证成功发送的最大序号，因此base应该永远保持在接受的号码中最大的值，即：
```c
modifyWaitingQueue(struct Queue *queue,struct NumBlock *numBlock,int updateSeqNum){
  updateSeqNum=updateSeqNum>numBlock->base?updateSeqNum:numBlock->base;
  queue->batchPop(queue,updateSeqNum-numBlock->base);
  numBlock->base=updateSeqNum;
}
```
另外在缓冲区的实现上使用了循环队列来控制，加入了函数式编程思想提高代码简洁度，实现如下：
```c
/*
*   Defination for data type queue
*/
struct Queue;
#define MAXWAITINGNUM 10
int add(struct Queue *queue,struct pkt packet);
int isFull(struct Queue queue);
int isEmpty(struct Queue queue);
struct pkt pop(struct Queue *queue);
struct pkt top(struct Queue queue);
void batchPop(struct Queue *queue,int num);
void apply(struct Queue *queue,void (*funcPtr)(struct pkt* packet));
struct Queue{
  int start;
  int end;
  struct pkt pktList[MAXWAITINGNUM];
  int (*add)(struct Queue*,struct pkt);
  int (*isFull)(struct Queue);
  int (*isEmpty)(struct Queue);
  struct pkt (*pop)(struct Queue*);
  struct pkt (*top)(struct Queue);
  void (*batchPop)(struct Queue*,int);
  void (*apply)(struct Queue*,void (*)(struct pkt*));
};
void QueueInit(struct Queue *queue){
  queue->start=0;
  queue->end=0;
  queue->add=&add;
  queue->isFull=&isFull;
  queue->isEmpty=&isEmpty;
  queue->pop=&pop;
  queue->top=&top;
  queue->batchPop=&batchPop;
  queue->apply=&apply;
}
int isFull(struct Queue queue){
  return (queue.end+1)%MAXWAITINGNUM==queue.start;
}
int add(struct Queue *queue,struct pkt packet){
  if(queue->isFull(*queue)){
    return 0;
  }
  queue->pktList[queue->end]=packet;
  queue->end=(queue->end+1)%MAXWAITINGNUM;
  return 1;
}
int isEmpty(struct Queue queue){
  return queue.end==queue.start;
}
struct pkt pop(struct Queue *queue){
  int tmp=queue->start;
  queue->start=(queue->start+1)%MAXWAITINGNUM;
  return queue->pktList[tmp];
}
struct pkt top(struct Queue queue){
  return queue.pktList[queue.start];
}
void batchPop(struct Queue *queue,int num){
  queue->start=(queue->start+num)%MAXWAITINGNUM;
}
void apply(struct Queue *queue,void (*funcPtr)(struct pkt *packet)){
  int start_=queue->start;
  while(start_!=queue->end){
    funcPtr(&queue->pktList[start_]);
    start_=(start_+1)%MAXWAITINGNUM;
  }
}
```

## :sparkling_heart:License

ITrans 采用 [木兰宽松许可证，第2版](https://license.coscl.org.cn/MulanPSL2), 可以自由拷贝和使用源码, 当做修改或分发时, 请遵守 [木兰宽松许可证，第2版](https://license.coscl.org.cn/MulanPSL2).

![License](https://img.shields.io/badge/license-MuLan-yellow)



Copyright :copyright:2023 IST

