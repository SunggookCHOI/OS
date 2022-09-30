### PODMAN 실습
# PODMAN 정보확인
podman info
[student@servera ~]$ podman login registry.lab.example.com

# Registry : 컨테이너 이미지 중앙 서버(서버 개념)
# Repository : Registry에서 컨테이너 이미지 저장 공간(서버 안의 저장 공간 개념)
# Tag : Repository에서 특정 이미지를 지정/구분할 때 사용 ; Repository 에 업로드되는 세부 이미지의 이름. 
  latest (가장 최근에 빌드한 이미지)

# 이미지 받기
podman pull registry.lab.example.com/ubi8/python-38:latest
[student@servera ~]$  podman images
REPOSITORY                               TAG         IMAGE ID      CREATED       SIZE
registry.lab.example.com/ubi8/python-38  latest      b335f517d3b5  4 months ago  892 MB

# 컨테이너 생성 ; 이름이 필수는 아니지만, 지정해서 관리하는걸 권장
podman create --name python38 IMAGE_REPO:TAG
[student@servera ~]$ podman create --name python38 registry.lab.example.com/ubi8/python-38:latest sleep infinity
[student@servera ~]$ podman ps -a
CONTAINER ID  IMAGE                                           COMMAND               CREATED         STATUS      PORTS       NAMES
9b90539d3172  registry.lab.example.com/ubi8/python-38:latest  /bin/sh -c $STI_S...  26 seconds ago  Created                 python38
[student@servera ~]$ podman start python38


# 사용중인 이미지는 못지운다.
[student@servera ~]$ podman images
REPOSITORY                               TAG         IMAGE ID      CREATED       SIZE
registry.lab.example.com/ubi8/python-38  latest      b335f517d3b5  4 months ago  892 MB
[student@servera ~]$ podman rmi registry.lab.example.com/ubi8/python-38:latest 
Error: Image used by 34a917927337559690170a81b0df4d827aedc56b219e9f1f0a018125b6ccde7a: image is in use by a container

[student@servera ~]$ podman ps
CONTAINER ID  IMAGE                                           COMMAND         CREATED         STATUS             PORTS       NAMES
4510e75890e9  registry.lab.example.com/ubi8/python-38:latest  sleep infinity  15 minutes ago  Up 15 minutes ago              python38
34a917927337  registry.lab.example.com/ubi8/python-38:latest  sleep infinity  12 minutes ago  Up 12 minutes ago              python38_2
[student@servera ~]$ podman stop python38 또는 podman stop 4510e75890e9
[student@servera ~]$ podman stop python38_2
# podman rm -f 옵션주면 실행중인 컨테이너 삭제 가능(권장하지는 않는다)
[student@servera ~]$ podman rm python38
4510e75890e985e903966a0164a0063f7d9b856f241a1c25dd0b5aab5b4721d5
[student@servera ~]$ podman rm python38_2
34a917927337559690170a81b0df4d827aedc56b219e9f1f0a018125b6ccde7a
[student@servera ~]$ podman ps -a
CONTAINER ID  IMAGE       COMMAND     CREATED     STATUS      PORTS       NAMES

[student@servera ~]$ podman images
REPOSITORY                               TAG         IMAGE ID      CREATED       SIZE
registry.lab.example.com/ubi8/python-38  latest      b335f517d3b5  4 months ago  892 MB
[student@servera ~]$ podman rmi registry.lab.example.com/ubi8/python-38:latest 
Untagged: registry.lab.example.com/ubi8/python-38:latest
Deleted: b335f517d3b508a101a197b926bc501f2139112269e8b7432ba304a21b81343f
[student@servera ~]$ podman images
REPOSITORY  TAG         IMAGE ID    CREATED     SIZE



[student@servera ~]$ podman network ls
NETWORK ID    NAME        DRIVER
2f259bab93aa  podman      bridge
[student@servera ~]$ podman inspect podman
[
     {
          "name": "podman",
          "id": "2f259bab93aaaaa2542ba43ef33eb990d0999ee1b9924b557b7be53c0b7a1bb9",
          "driver": "bridge",
          "network_interface": "podman0",
          "created": "2022-09-30T02:00:27.914134361-04:00",
          "subnets": [
               {
                    "subnet": "10.88.0.0/16",
                    "gateway": "10.88.0.1"
               }
          ],
          "ipv6_enabled": false,
          "internal": false,
          "dns_enabled": false,
          "ipam_options": {
               "driver": "host-local"
          }
     }
]



[student@servera ~]$ mkdir /hoem/student/db_data^C
[student@servera ~]$ podman unshare chown 27:27 /thome/student/db_data^C
[student@servera ~]$ podman run -d --name db01 -e MYSQL_USER=student -e MYSQL_PASSWORD=student -e MYSQL_DATABASE=dev_data -e MYSQL_ROOT_PASSWORD=redhat -v /home/student/db_data/:/var/lib/mysql:Z -p 13306:3306 --network db_net registry.lab.example.com/rhel8/mariadb-105
bca6488103e9be6580cf82f96ed88011c9c88cc90cb51010de7e4c93081315f7
