##1、tcp的流是分段的、由ip分组发送    

http传输一段报文时，会以流的形式通过一个打开的tcp连接按序传输。tcp接收到流的时候后，会将流砍成被称作段的小数据块，并将段封装在ip分组中，每个ip分组都包括 

- ip分组通常20字节  
  包含源和目的ip地址，长度和其他标记
- tcp段首部通常20个字节
  包含tcp端口号和tcp控制标记以及用于数据排序和完整性检查的一些数字值
- tcp数据块0或多个字节

##2、tcp传输的正确运行

任意时刻计算机都可以同时保持多个tcp连接，他们通过端口来保持正确运行。并用四个属性来确定一条唯一连接：

- 源ip
- 源端口
- 目的ip
- 目的端口


