构建出的镜像，可运行为tracker server也可作为storage server。

---------------------------------------------------------------------------

#构建镜像并推送到私有仓库

docker build -t 192.168.25.124:5000/fastdfs:5.11 .

docker push 192.168.25.124:5000/fastdfs:5.11

---------------------------------------------------------------------------
#1)在192.168.25.124上运行tracker server

docker run -d \\

--name fdfs_tracker \\

-v /home/fastdfs/tracker:/export/fastdfs/tracker \\

--net=host \\

-e TRACKER_BASE_PATH=/export/fastdfs/tracker \\

-e TRACKER_PORT=22123 \\

192.168.25.124:5000/fastdfs:5.11  \\

sh /usr/local/src/tracker.sh

---------------------------------------------------------------------------

#2)在192.168.25.124上运行storage server



docker run -d --name fdfs_storage \\

-v /home/fastdfs/storage:/export/fastdfs/storage \\

--net=host \\

-e STORAGE_PORT=23001 \\

-e STORAGE_BASE_PATH=/export/fastdfs/storage \\

-e STORAGE_PATH0=/export/fastdfs/storage \\

-e TRACKER_SERVER=192.168.25.124:22123,192.168.25.125:22123 \\

-e GROUP_COUNT=1 \\

-e HTTP_SERVER_PORT=8081 \\

-e GROUP_NAME=group1 \\

192.168.25.124:5000/fastdfs:5.11 \\

sh /usr/local/src/storage.sh

---------------------------------------------------------------------------

#3)在192.168.25.125上运行tracker server



docker run -d \\

--name fdfs_tracker \\

-v /home/fastdfs/tracker:/export/fastdfs/tracker \\

--net=host \\

-e TRACKER_BASE_PATH=/export/fastdfs/tracker \\

-e TRACKER_PORT=22123 \\

192.168.25.124:5000/fastdfs:5.11  \\

sh /usr/local/src/tracker.sh

---------------------------------------------------------------------------

#4)在192.168.25.125上运行storage server



docker run -d --name fdfs_storage \\

-v /home/fastdfs/storage:/export/fastdfs/storage \\

--net=host \\

-e STORAGE_PORT=23001 \\

-e STORAGE_BASE_PATH=/export/fastdfs/storage \\

-e STORAGE_PATH0=/export/fastdfs/storage \\

-e TRACKER_SERVER=192.168.25.124:22123,192.168.25.125:22123 \\

-e GROUP_COUNT=1 \\

-e HTTP_SERVER_PORT=8081 \\

-e GROUP_NAME=group1 \\

192.168.25.124:5000/fastdfs:5.11 \\

sh /usr/local/src/storage.sh

---------------------------------------------------------------------------

进入容器或者在宿主机上查看日志：发现tracker server有两个，其中leader tracker server是192.168.25.124:22123

[root@cool logs]# ip addr show | grep 192.168.25.125

    inet 192.168.25.125/24 brd 192.168.25.255 scope global noprefixroute ens33
    
[root@cool logs]# cd /home/fastdfs/storage/logs

[root@cool logs]# tail storaged.log 

data path: /export/fastdfs/storage/data, mkdir sub dir done.

[2019-05-09 01:54:43] INFO - file: storage_param_getter.c, line: 191, use_storage_id=0, id_type_in_filename=ip, storage_ip_changed_auto_adjust=1, store_path=0, reserved_storage_space=10.00%, use_trunk_file=0, slot_min_size=256, slot_max_size=16 MB, trunk_file_size=64 MB, trunk_create_file_advance=0, trunk_create_file_time_base=02:00, trunk_create_file_interval=86400, trunk_create_file_space_threshold=20 GB, trunk_init_check_occupying=0, trunk_init_reload_from_binlog=0, trunk_compress_binlog_min_interval=0, store_slave_file_use_link=0

[2019-05-09 01:54:43] INFO - file: storage_func.c, line: 257, tracker_client_ip: 192.168.25.125, my_server_id_str: 192.168.25.125, g_server_id_in_filename: 2098833600

[2019-05-09 01:54:43] INFO - file: tracker_client_thread.c, line: 310, successfully connect to tracker server 192.168.25.125:22123, as a tracker client, my ip is 192.168.25.125

[2019-05-09 01:54:43] INFO - file: tracker_client_thread.c, line: 1947, tracker server: #0. 192.168.25.124:22123, my_report_status: -1

[2019-05-09 01:54:44] INFO - file: tracker_client_thread.c, line: 310, successfully connect to tracker server 192.168.25.124:22123, as a tracker client, my ip is 192.168.25.125

[2019-05-09 01:54:44] INFO - file: tracker_client_thread.c, line: 1947, tracker server: #0. 192.168.25.124:22123, my_report_status: -1

[2019-05-09 01:54:44] INFO - file: tracker_client_thread.c, line: 1263, tracker server 192.168.25.124:22123, set tracker leader: 192.168.25.124:22123

[2019-05-09 01:54:44] INFO - file: storage_sync.c, line: 2732, successfully connect to storage server 192.168.25.124:23001



