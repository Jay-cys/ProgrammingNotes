
# 1 原始的PADR buffer
![image.png](.assets/1593744745892-a560b1ed-79d4-4e7e-9ed1-4a61e94ac674.png)

- **buffer length**
```cpp
((u16 *) frameData_)[0] = length;     //buffer的长度 - 4
```

- **buffer offset**
```cpp
((u16 *) frameData_)[1] = ifId; //(ifId << 1) | 1;  //那个接口发来的消息，interface id
```


# 2 **Q921 帧结构**

## 2.1 address field
```c
//address field
frameData_[4] = (sapi << 2)  | (CRon << 1) | 0; //SAPI和CRon的值
frameData_[5] = (tei << 1)  | 1;  //TEI的值
//end address field
```
![image.png](.assets/1593744879288-afdd2bca-acb9-4811-9db5-f4e16fa4a5c9.png)

## 2.2 **control field**
```cpp
//control field
frameData[6] = ns << 1;         //NS的值
frameData[7] = (frameData[7] & 0x01) | (nr << 1);  //NR的值
//end control field
```
![image.png](.assets/1593744909602-0aa6c451-d328-4e06-8f46-2149c93696ef.png)

# 3 Q931帧结构

## 3.1 **protocol discriminator**
```cpp
frameData[8] = 0x08;  //protocol discriminator
```
![image.png](.assets/1593745144507-7a363cb0-4b9b-4987-b5b5-e3b2e56547fc.png)

## 3.2 **call reference**
```cpp
frameData[9] = 0x02;  //call reference length, 表示下面哪种结构
frameData[10] = 0x00; // reference flag： 0 means the message is sent from the side that originates the call reference
frameData[11] = call_reference;
```
![image.png](.assets/1593745191669-9bde0088-cbce-4b33-ac34-e68d72e97f09.png)

## 3.3 **message type**
```cpp
frameData[12] = msg_type;
```
![image.png](.assets/1593745220123-aaa28589-ff01-4208-894a-c7e3b4d49d94.png)

## 3.4 **其他IE**
从_frameData[13]_开始，属于其他IE，不同message type，对应不同IE, 通过element identifier来判断属于什么IE，通过每个octet（8bit）的第8位判断该IE的内容是否结束**(1表示结束，0表示还有下一个octet）**。 下面介绍常见的called number和calling number的buffer结构：

![image.png](.assets/1593745392313-260276eb-c6c5-4068-badc-11436607e212.png)

