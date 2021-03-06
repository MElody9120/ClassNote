# Chapter3 数据链路层

---

考试内容
- 理解数据链路层的三个基本功能-组帧、透明传输和差错控制
- 理解介质访问控制协议，重点掌握CSMA、CSMA/CD、CSMA/CA的基本原理
- 掌握广域网协议中的HDLC协议和PP协议的特点与区别
- 理解网桥尤其是透明网桥的工作原理，注意网桥和中继器的对比

本章命题以客观题为主，重点考察基于滑动窗口的流量控制，随机访问介质访问控制协议，以太网帧格式往往会与其他章节联合命题

---

## 知识结构

本章的知识结构图如下mermaid图所示

```mermaid
graph RL
a(数据链路层) --> b1(数据链路层的基本问题)
b1 --> c1(组帧)
b1 --> c2(透明传输)
c2 --> d1(字符填充)
c2 --> d2(比特填充)
b1 --> c3(差错控制)
c3 --> d3(奇偶校验)
d3 --> e1(常用的校验方法)
c3 --> d4(CRC循环校验)
d4 --> e1
a --> b2(流量控制)
b2 --> c4(停止等待滑动窗口)
b2 --> c5(SR)
b2 --> c6(GBN)
a --> b3(介质访问控制)
b3 --> c7(静态介质访问控制)
b3 --> c8(轮询访问介质访问控制)
b3 --> c9(争用型介质控制方法)
c9 --> d5(ALOHA)
c9 --> d6(CSMA)
c9 --> d7(CSMA/CD)
c9 --> d8(CSMA/CA)
a --> b4(局域网的数据链路层)
b4 --> c10(局域网的基本概念)
b4 --> c11(802.3的数据链路层)
b4 --> c12(以太网的工作原理)
b4 --> c13(以太网的MAC层)
b4 --> c14(高速以太网)
a --> b5(广域网的数据链路层)
b5 --> c15(PPP协议)
b5 --> c16(HDLC协议)
a --> b6(数据链路层设备)
b6 --> c17(网桥)
c17 --> d9(透明网桥)
c17 --> d10(源路由网桥)
b6 --> c18(交换机)
```