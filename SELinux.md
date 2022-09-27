
# SElinux 상태확인
getenforce

# SElinux 상태변경
setenforce=enforcing|Permissive..

# SElinux 비활성화
grubby --update-kernel ALL --args selinux=0
# To revert back to SELinux enabled:
grubby --update-kernel ALL --remove-args selinux

# SElinux 정책확인
ls -lZ, ps -efZ, ...
보통 type의 이름 비교

#type context 임시 변경
chcon -t CONTEXT FILE
chcon -t tmp_t test1.txt

# SElinux 정책 확인
semanage fcontext -l

# type context 영구적 변경
semanage fcontext -a -t 'httpd_sys_content_t' '/FILE/PATH(/.*)?'
restorecon -FRv .(해당 디렉토리에서)


# SELinux 설정 1. run time 설정 2. Permanent 설정
# 1. run time 설정 -> 재부팅시 permanent 옵션으로 덮어써진다.
setsebool httpd_enable_homedirs on
# 2. Permanent 설정
setsebool -P httpd_enable_homedirs on

#  변경된 항목만 확인
semanage boolean -l -C


#SELinux Trouble Shooting
1. 정책확인
2. Enforcing에서 Permissive로 바꿔본다 -> 해결되면 SELinux 문제. 그래도 안되면 서비스 문제일 확률이 높다.
3. /var/log/audit/audit.log 로그 확인(setroubleshoot-server 패키지 필요) -> syslog에도 기록됨(요약본)
4. sealert -l UUID
