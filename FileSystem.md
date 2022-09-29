### 파티션
# 파티션 정보 확인
parted /dev/vda

# 파티셔닝
parted /dev/vdb mklabel gpt[gpt]/msdos[mbr]

parted /dv/vdb mkpart primary xfs 2048s[새 파티셔닝 시작할 디스크 섹터] 1000MB[파티셔닝 끝낼 디스크 섹터]
# 커널에 정보 업데이트
udevadm settle

# 파일시스템 초기화
mkfs -t FS[xfx,ext4,..] /dev/vdc1 
# mount
mount -t FS PARTITION_DEVICE MOUNT_POINT
mount -t ext4 /dev/vdc1 /mnt/data1
# 확인
mount
#umount
umount MOUNT_POINT
umount DEVICE

# 영구 마운트
/etc/fstab

### SWAP
# swap 영역 만들때는 FS type linux-swap 으로 설정
# 스왑 공간 포맷
mkswap /dev/vdb2
swapon /dev/vdb2
swapoff
# 영구설정 /etc/fstab
UUID  swap  swap  defaults  0 0

### 스토리지 스택 관리 LVM
파티셔닝의 경우, 기존 디스크를 모두 사용하면 증설이 어렵다.
-> 유연성 향상
PE : PV에서 데이터 입출력 단위(기본 4M)
LE : LV에서 데이터 입출력 단위(기본적으로 PE size와 동일)

pvcreate DEVICE
vgcreate VG_NAME DEVICE [-s PE_Size]
lvcreate -n LV_NAME -L 용량 -l LE개수 VG_NAME

# LVM LV PATH
/dev/VG_NAME/LV_NAME
/dev/mapper/VG_NAME-LV_NAME

# 논리 볼륨 확장
lvextend -L +500M /dev/vg01/lv01
# FS 확장
xfs_growfs|resize2fs /dev/vg01/lv/01

# FS 확장을 포함한 LVM 확장 방법
lvextend -L(-l) +늘릴용량|최종용량 LV_NM -r


# swap 공간 논리 볼륨 확장
swapoff -v /dev/vg01/swap
lvextend -L +300M /dev/vg01/swap
mkswap /dev/vg01/swap
swapon /dev/vg01/swap

# LVM 스토리지 제거는 역순
umount -> lvremove -> vgremove -> pvremove


### Stratis
# 풀 생성
stratis pool create POOL DEVICE
# 풀 추가
stratis pool add-data pool1 /dev/vdc
# 풀 확인
stratis pool list
stratis blockdev list pool1
# FS 생성
stratis filesystem create pool1 FS_NM
stratis filesystem list
# FS UUID 확인
lsblk --output=UUID /dev/stratis/pool1/fs1
# /etc/fstab  ** defaults 뒤의 옵션 매우 중요
UUID=7eed0ded-9dfb-4dde-befe-d9b415015069       /dir1   xfs     defaults,x-systemd.requires=stratisd.service    0       0
mkdir /dir1
mount /dir1


### NFS
mkdir /mountpoint
mount -t nfs -o rw,sync server:/export /mountpoint
#/etc/fstab
SERVER:/EXPORT/ /mountpoint nfs rw,sync 0 0

  
## serverb 의 FS를 servera에서 사용
# server 설정
vim /etc/exports.d/public.exports
/FS *(rw,no_root_squash)
[root@serverb exports.d]# systemctl start nfs-server.service
[root@serverb exports.d]# systemctl enable nfs-server.service
# 방화벽 정책 설정
[root@serverb exports.d]# firewall-cmd --add-service=nfs --permanent
[root@serverb exports.d]# semanage fcontext -a -t 'public_content_rw_t' '/netstorage(/.*)?'
[root@serverb netstorage]# restorecon -FRv /netstorage/
[root@serverb exports.d]# chmod 2770 /netstorage
[root@serverb netstorage]# exportfs -r

# Client 설정
[root@servera ~]# mount -t nfs serverb:/netstorage/test1 /mnt/manual
[root@servera ~]# vim /etc/fstab
serverb:/netstorage/test1       /mnt/manual     nfs     rw,sync 0       0


## Auto FS, Auto mount
필요할때만 연결하여 자원 절약

# autofs 마운트 매핑 방식
1. 직접 매핑

2. 간접 매핑
  a. 1:1
  b. 와일드카드(다대다)
 
# NFS automounter(autofs) 구성단계
1. 패키지 설치
  dnf install autofs nfs-utils
2. 마스터맵 작성(마운트 방식, 매핑파일 선언)
  vim /etc/auto.master.d/MASTER_MAP_FILE.autofs
3. 매핑파일 작성
  vim /etc/auto.MAP_FILE_NAME
3-1. (직접매핑의 경우에만) 로컬 마운트 포인트 생성
  mkdir /mnt/docs
4. 서비스 시작 및 화렁화(영구설정)
  systemctl enable --now autofs
  
## 직접매핑
vim /etc/auto.master.d/direct.autofs
  /-      /etc/auto.direct  (설정파일명)
vim /etc/auto.direct
  /mnt/direct     -rw,sync        serverb:/shares/management

[root@servera ~]# systemctl start autofs.service
[root@servera ~]# systemctl enable autofs.service
# 위 두줄 = systemctl enable --now autofs 한줄로도 가능

## 1:1 간접매핑
# 마스터맵 작성
[root@servera direct]# vim /etc/auto.master.d/indirect1.autofs
  /mnt/indirect1  /etc/auto.indirect1
# 매핑 파일 작성
[root@servera direct]# vim /etc/auto.indirect1
  operation	-rw,sync	serverb:/shares/operation
  production	-rw,sync	serverb:/shares/production
# 간접매핑은 별도로 마운트포인트 생성 안해도 된다.
[root@servera direct]# systemctl restart autofs
# 아직 operation이 없다.
[root@servera mnt]# cd indirect1/
[root@servera indirect1]# ls
# /mnt/indirect1 까지만 마운트되어있다고 나온다
[root@servera indirect1]# mount | grep mnt
/etc/auto.direct on /mnt/direct type autofs (rw,relatime,fd=17,pgrp=1764,timeout=300,minproto=5,maxproto=5,direct,pipe_ino=27989)
serverb:/shares/management on /mnt/direct type nfs4 (rw,relatime,sync,vers=4.2,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=172.25.250.10,local_lock=none,addr=172.25.250.11)
/etc/auto.indirect1 on /mnt/indirect1 type autofs (rw,relatime,fd=23,pgrp=1764,timeout=300,minproto=5,maxproto=5,indirect,pipe_ino=27997)

# 없는데 들어가진다(최초에)
[root@servera indirect1]# cd /mnt/indirect1/operation
# 이제는 /mnt/indirect1/operation 까지 마운트 되어있다.
[root@servera operation]# mount | grep mnt
/etc/auto.direct on /mnt/direct type autofs (rw,relatime,fd=17,pgrp=1764,timeout=300,minproto=5,maxproto=5,direct,pipe_ino=27989)
serverb:/shares/management on /mnt/direct type nfs4 (rw,relatime,sync,vers=4.2,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=172.25.250.10,local_lock=none,addr=172.25.250.11)
/etc/auto.indirect1 on /mnt/indirect1 type autofs (rw,relatime,fd=23,pgrp=1764,timeout=300,minproto=5,maxproto=5,indirect,pipe_ino=27997)
serverb:/shares/operation on /mnt/indirect1/operation type nfs4 (rw,relatime,sync,vers=4.2,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=172.25.250.10,local_lock=none,addr=172.25.250.11)


## 와일드카드 간접매핑 -> 위처럼 일일히 다 지정하기 귀찮다.
# 설정 초기화
[root@servera auto.master.d]# mv * /etc/auto.master.d/tmp
[root@servera auto.master.d]# systemctl restart autofs
# 아무것도 없다.
[root@servera auto.master.d]# mount | grep /mnt
# 마스터맵 설정
[root@servera auto.master.d]# vim /etc/auto.master.d/indirect.autofs
/mnt/indirect   /etc/auto.indirect
# 매핑파일 생성
[root@servera auto.master.d]# vim /etc/auto.indirect
*       -rw,sync        serverb:/shares/&
[root@servera mnt]# systemctl restart autofs
# /mnt/indirect 디렉토리 생김
# 1:1 
[root@servera mnt]# cd indirect
[root@servera indirect]# ls

[root@servera indirect]# mount | grep /mnt
/etc/auto.indirect on /mnt/indirect type autofs (rw,relatime,fd=17,pgrp=1895,timeout=300,minproto=5,maxproto=5,indirect,pipe_ino=29409)
[root@servera indirect]# cd operation
[root@servera operation]# mount | grep /mnt
/etc/auto.indirect on /mnt/indirect type autofs (rw,relatime,fd=17,pgrp=1895,timeout=300,minproto=5,maxproto=5,indirect,pipe_ino=29409)
serverb:/shares/operation on /mnt/indirect/operation type nfs4 (rw,relatime,sync,vers=4.2,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=172.25.250.10,local_lock=none,addr=172.25.250.11)
