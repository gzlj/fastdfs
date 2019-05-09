#fastdfs_in_docker
<p>
当前仅支持1个tracker，一组storage的部署方式。每个storage上都装有nginx，提供http服务的端口为8080。可按如下的方式对文件进行http访问：http://192.168.83.176:8080/group1/M00/00/00/wKhTsFkRlTOAQqm6ABvFny1kWJ4443.mp4
</p>
<p>
假设存在安装场景：1个tracker；1组storage，包括两个storage；
tracker和其中一个storage安装在192.168.83.176上，另一个storage安装在192.168.83.177。
部署拓扑如下：
</p>
<pre><code>
            tracker(83.176)
              /      \
             /        \
------------/          \-------------
| group1   /            \           |
|         /              \          |
|        /                \         |
|     storage1         storage2     |
|     (83.176)         (83.177)     |
|                                   |
------------------------------------
</code></pre>
<p>
以上述安装场景为例，工程使用方法如下：
<ol>
<li>在宿主机上安装docker</li>
<li>下载工程到宿主机上，假定存储到宿主机的目录A</li>
<li>进入目录A，执行命令
   docker build -t zjg23/fastdfs:1.0 .
   构建镜像</li>
<li>在192.168.83.176上运行tracker<br />
   docker run -d --name fdfs_tracker --net=host -e TRACKER_BASE_PATH=/export/fastdfs/tracker zjg23/fastdfs:1.0 sh /usr/local/src/tracker.sh<br />  
   在192.168.83.176上运行storage<br />
   docker run -d --name fdfs_storage --net=host -e STORAGE_BASE_PATH=/export/fastdfs/storage  -e STORAGE_PATH0=/export/fastdfs/storage -e TRACKER_SERVER=192.168.83.176:22122 -e GROUP_COUNT=1 -e HTTP_SERVER_PORT=8080 zjg23/fastdfs:1.0 sh /usr/local/src/storage.sh<br />     
   在192.168.83.177上运行storage<br />     
   docker run -d --name fdfs_storage_2 --net=host -e STORAGE_BASE_PATH=/export/fastdfs/storage  -e STORAGE_PATH0=/export/fastdfs/storage -e TRACKER_SERVER=192.168.83.176:22122 -e GROUP_COUNT=1 -e HTTP_SERVER_PORT=8080 zjg23/fastdfs:1.0 sh /usr/local/src/storage.sh</li>
</ol>
至此，fastdfs安装完成。
</p>

<p>
todo:
<ol>
<li>运行参数中添加宿主机和容器的存储映射，使得fastdfs使用宿主机的存储--20170523完成<br />
命令样例：<br />
docker run -d --name fdfs_tracker  -v /home/fastdfs/tracker:/export/fastdfs/tracker --net=host -e TRACKER_BASE_PATH=/export/fastdfs/tracker zjg23/fastdfs:1.0 sh /usr/local/src/tracker.sh<br />

docker run -d --name fdfs_storage  -v /home/fastdfs/storage:/export/fastdfs/storage --net=host -e STORAGE_BASE_PATH=/export/fastdfs/storage -e STORAGE_PATH0=/export/fastdfs/storage -e TRACKER_SERVER=192.168.83.176:22122 -e GROUP_COUNT=1 -e HTTP_SERVER_PORT=8080 zjg23/fastdfs:1.0 sh /usr/local/src/storage.sh<br />
</li>
<li>运行参数中添加资源限制（cpu，内存等）</li>
<li>docker镜像操作系统的参数调优</li>
<li>docker镜像文件大小是否可优化</li>
<li>storage多组安装--20170525完成<br/>
举例：<br/>
192.168.83.177--》tracker<br/>
192.168.83.177,192.168.83.176-->group1 storage<br/>
192.168.83.170,192.168.83.172-->group2,storage<br/>
<br/>
1、所有机器上执行：<br/>
mkdir -p /home/fastdfs/{tracker,storage}<br/>
docker build -t zjg23/fastdfs:2.0 .<br/> 
<br/>
2、构建tracker,177上执行：<br/>
docker run -d --name fdfs_tracker  -v /home/fastdfs/tracker:/export/fastdfs/tracker --net=host -e TRACKER_BASE_PATH=/export/fastdfs/tracker -e TRACKER_PORT=22123 zjg23/fastdfs:2.0 sh /usr/local/src/tracker.sh<br/>
<br/>
3、构建storage<br/>
3.1 177上执行：<br/>
docker run -d --name fdfs_storage -v /home/fastdfs/storage:/export/fastdfs/storage --net=host -e STORAGE_PORT=23001 -e STORAGE_BASE_PATH=/export/fastdfs/storage -e STORAGE_PATH0=/export/fastdfs/storage -e TRACKER_SERVER=192.168.83.177:22123 -e GROUP_COUNT=2 -e HTTP_SERVER_PORT=8080 -e GROUP_NAME=group1 zjg23/fastdfs:2.0 sh /usr/local/src/storage.sh<br/>
3.2 176上执行：<br/>
docker run -d --name fdfs_storage -v /home/fastdfs/storage:/export/fastdfs/storage --net=host -e STORAGE_PORT=23001 -e STORAGE_BASE_PATH=/export/fastdfs/storage -e STORAGE_PATH0=/export/fastdfs/storage -e TRACKER_SERVER=192.168.83.177:22123 -e GROUP_COUNT=2 -e HTTP_SERVER_PORT=8080 -e GROUP_NAME=group1 zjg23/fastdfs:2.0 sh /usr/local/src/storage.sh<br/>
3.3 170上执行：<br/>
docker run -d --name fdfs_storage -v /home/fastdfs/storage:/export/fastdfs/storage --net=host -e STORAGE_PORT=23001 -e STORAGE_BASE_PATH=/export/fastdfs/storage -e STORAGE_PATH0=/export/fastdfs/storage -e TRACKER_SERVER=192.168.83.177:22123 -e GROUP_COUNT=2 -e HTTP_SERVER_PORT=8080 -e GROUP_NAME=group2 zjg23/fastdfs:2.0 sh /usr/local/src/storage.sh<br/>
3.4 172上执行：<br/>
docker run -d --name fdfs_storage -v /home/fastdfs/storage:/export/fastdfs/storage --net=host -e STORAGE_PORT=23001 -e STORAGE_BASE_PATH=/export/fastdfs/storage -e STORAGE_PATH0=/export/fastdfs/storage -e TRACKER_SERVER=192.168.83.177:22123 -e GROUP_COUNT=2 -e HTTP_SERVER_PORT=8080 -e GROUP_NAME=group2 zjg23/fastdfs:2.0 sh /usr/local/src/storage.sh<br/>

</li>
<li>工程下Dockfile文件的介绍，即安装思路</li>
<li>tracker多点安装</li>
</ol>
</p>