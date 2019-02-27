#### kubernetes pod实现自动水平伸缩-创建HPA

kubernetes经常遇到资源紧张需要扩容的情况，这时可以手动的去对deploy或者replica set进行手动命令扩容，命令

```shell
# n为副本数
kubectl scale deployment -n namespace --replicas=n

```

这样可以通过扩容deployment或者rs的数量来达到扩容pod的目标。

但是这种扩容方式在经过了应用峰值后，如果想释放资源空间，还需要手动的去将副本数降低，这不能达到释放人工的最大目标。再后来的kubernetes版本中，引进了Horizontal Pod Autoscaler（HPA）这个概念，pod可以根据cpu资源进行自动的扩缩容，依据heapster和matric-server的监控数据。

- autoscaling/v1: CPU 
- autoscaling/v2alpha1 内存 自定义metrics 多metrics组合: 根据每个metric的值计算出scale的值，并将最大的那个值作为扩容的最终结果

###### 创建hpa

```shell
apiVersion: version/v1
# 创建hpa的yaml类型
kind: HorizontalPodAutoscaler
metadata:
  name: public-visplat-instance
  namespace: test
spec:
  scaleTargetRef:
  # autoscaling/v1: CPU 
  # autoscaling/v2alpha1 内存 自定义metrics 多metrics组合: 根据每个metric的值计算出scale的值，并将最大的那个值作为扩容的最终结果

    apiVersion: v1
    # deployment 或者 replicacontroller
    kind: deployment
    name: public-visplat-instance
    # 最小副本数 最大副本数
  minReplicas: 1
  maxReplicas: 10
  #扩容阈值
  targetCPUUtilizationPercentage: 50
```



也可以根据命令进行hpa创建

```shell
kubectl autoscale deployment instancename -n namespace --cpu-percent=x% --min=a --max=b
```



查看hpa

```
kubectl get hpa -n namespace
```



![](/Users/rqw1991/Devops-LYZ/kubernetes/img/通过hpa进行扩容.jpeg)

![img](/Users/rqw1991/Devops-LYZ/kubernetes/img/hpa动态扩容200%.jpeg)