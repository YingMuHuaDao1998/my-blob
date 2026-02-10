### 三个属性设置

- requirepass

  可以不进行设置（忽略安全问题），如果设置，主从设置一致。

  ![image-20240226175637689](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20240226175637689.png)

- masterauth

  可以不进行设置（内网访问无需设置），如果设置，主从设置一致。

  ![image-20240226175711443](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20240226175711443.png)

- repl-disable-tcp-nodelay

  ![image-20240226175736780](/Users/xuchaoyue/Library/Application Support/typora-user-images/image-20240226175736780.png)

该属性用于设置是否禁用 TCP 特性 tcp-nodelay。设置为 yes 则禁用 tcp-nodelay，此时master 与 slave 间的通信会产生延迟，但使用的 TCP 包数量会较少，占用的网络带宽会较小。相反，如果设置为 no，则网络延迟会变小，但使用的 TCP 包数量会较多，相应占用的网络带宽会大。 
tcp-nodelay：为了充分复用网络带宽，TCP 总是希望发送尽可能大的数据块。为了达到该目的，TCP 中使用了一个名为 Nagle 的算法。 
Nagle 算法的工作原理是，网络在接收到要发送的数据后，并不直接发送，而是等待着数据量足够大（由 TCP 网络特性决定）时再一次性发送出去。这样，网络上传输的有效数据
比例就得到了大大提升，无效数据传递量极大减少，于是就节省了网络带宽，缓解了网络压
力。 
tcp-nodelay 则是 TCP 协议中 Nagle 算法的开头。 