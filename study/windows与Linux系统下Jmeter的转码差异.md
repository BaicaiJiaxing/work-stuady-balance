背景：执行银行缴费压测过程中通过csv data cofig 读取UTF-8编码文件，windows下和linux下发送到服务端的编码格式不同。windows发送的GBK编码、Linux发送的UTF-8编码。
通过AI得知（有待求证）：​

| 系统      | 默认编码       | Java默认编码 | 文本文件常见编码                 |
| ------- | ---------- | -------- | ------------------------ |
| Windows | GBK/GB2312 | GBK      | GBK, UTF-8 with BOM      |
| Linux   | UTF-8      | UTF-8    | UTF-8, UTF-8 without BOM |
- Windows系统​**​：Java默认使用GBK编码，JMeter自动将UTF-8文件内容转换为GBK编码发送
- ​**​Linux系统​**​：Java默认使用UTF-8编码，JMeter直接以UTF-8发送，没有进行必要的编码转换
基于以上给出解决方案：
1.自定义TCPClientGBK，将报文以GBK编码后发送到服务端，结果上看方法可行；