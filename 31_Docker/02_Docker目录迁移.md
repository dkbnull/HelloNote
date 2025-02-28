我们在生产环境中安装Docker时，默认的安装目录是**/var/lib/docker**，而通常情况下，规划给系统盘的目录一般为50G，该目录是比较小的，一旦容器过多或容器日志过多，就可能出现Docker无法运行的情况，所以我们进行Docker目录迁移

常用方法如下：

软连接方式

~~~sh
#停止docker服务
systemctl stop docker

#创建备份目录，防止迁移失败
cp -r /var/lib/docker /var/lib/docker-bak

#迁移到新目录
mv /var/lib/docker /data/docker

#创建软连接
ln -s /data/docker/ /var/lib/docker

#启动docker
systemctl start docker

#查看容器
docker ps
~~~

确保Docker正常启动，且容器能正常访问后，删除备份目录

~~~sh
rm -rf /var/lib/docker-bak
~~~



![image-20240423145312551](02_Docker%E7%9B%AE%E5%BD%95%E8%BF%81%E7%A7%BB.assets/image-20240423145312551.png)



---

CSDN：[https://blog.csdn.net/dkbnull/article/details/138476127](https://blog.csdn.net/dkbnull/article/details/138476127)

微信：[https://mp.weixin.qq.com/s/omghND4pV4KpxbaLIiAG4g](https://mp.weixin.qq.com/s/omghND4pV4KpxbaLIiAG4g)

---

