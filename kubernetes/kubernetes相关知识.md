kubernetes节点调度：

隔离节点，使节点不能在分配pod

kubectl cordon node

解除隔离节点

kubectl uncordon node

一般用来保护master节点，防止master节点被pod压垮。

