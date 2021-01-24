# 6.824 学习

该仓库直接从 MIT 6.824 教程上下载，未修改后上传 Beeter-yong 仓库下。首先对 Lab 2A 进行完成，再对 Lab 2B 进行完成。其他实验暂不考虑，只是为了更好的掌握 Raft 进行编程测试

Raft 设计中将分解成三个子问题：Leader election、Log replication、Safety

## Lab2A 完成领导者选举

Raft 论文中的 Figure2 对整个 Raft 设计规则进行了较为明确的规定，按理来说只要对 Figure2 进行实现就可以实现 Raft。我也将此图当成实现的目标，进而实现 Lab2A 和 Lab2B。

### Step01 Raft 变量声明

首先需要声明一种类型 Role，它用来标识每一个 Raft 实例的角色是 Leader、Candidate、Follower

根据 Figure2 中的 State 描述，就是一个 Raft 实例所应该具有的所有基本属性。但实际中要时刻记录 Raft 活动时长，我就用变量 LastActiveTime 作为 Raft 每一次响应的开始时间，则**当前 Raft
活动时间 = NowTime - LastActiveTime**

![State](https://cdn.jsdelivr.net/gh/Beeter-yong/pictures/imgTwo/20210120151120.png)

Raft 中的变量分为**持久化变量**和**易失性变量**，持久性变量是指当 Raft 实例宕机时重新恢复依然需要获取自己的历史记录，于是将这部分信息持久化下来；易失性变量就是宕机后恢复直接进行一个初始化操作即可继续参与 Raft 集群中。

我认为在 Lab2A 中只需要的变量是 **currentTerm、voteFor**（当前 term 下，该 server 将票投给了谁）

### Step02 RequestVote Rpc 变量声明和实现

![RequestVote Rpc](https://cdn.jsdelivr.net/gh/Beeter-yong/pictures/imgTwo/20210120153843.png)

#### 先设计 RPC 通信的结构体类型

请求 Rpc 即 RequestVoteArgs，在 Lab2A 阶段需要字段 Term、CandidateId

响应 RPC 即 RequestVoteReply，需要字段 Term、VoteGranted

#### 再设计实现 Vote RPC 实现的方法

即实现已经声明好的 RequestVote()：1. 对比请求 RPC 的 term 是否大于自身的 term；2. 查看 votedFor 自身是否已经对其他人投过票。

其中规则 2 中的对比是否最新日志不属于 Lab2A，暂且不考虑

### Step03 AppendEntries RPC 变量声明和实现

![AppendEntries RPC](https://cdn.jsdelivr.net/gh/Beeter-yong/pictures/imgTwo/20210120172524.png)

该部分在 Lab2A 中主要完成心跳信息，具体说来是当 Leader 发送心跳给 Servers 来维持自己的权威。但 6.824 没有给定这部分如何设计，可以模仿 VoteRpc 来进行设计

#### 设计 Rpc 通信结构体类型

请求 RPC 包括 term，LeaderId

响应 RPC 包括 term，success

#### 实现具体方法

当接收到一个**心跳信息**后，Server 如何处理，参考规则 1 即可

其如何通信参考 sendRequestVote() 方法，其中 src/labrpc 已经设计好通用的 RPC 通信方式，方法为 Call

### Step04 实现主动发起投票

Raft 实例都有想当 Leader 的冲动，设计一个函数 attempBecomeLeader() 来满足此需求

## 注意点

- 所有的方法实现都要考虑**锁**，不考虑底层 CPU 效率，上来就把函数锁住，但容易出现死锁情况，debug 时调整粒度
- MIT6.824 设计上要求 RPC 通信类型必须**大写字母开头**
