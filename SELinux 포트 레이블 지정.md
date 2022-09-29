### SELinux 포트 레이블 지정
SELinux가 실행중인경우, default port가 아닌 다른 포트를 사용중인경우 차단된다.
# 포트 레이블 확인
semanage port -l
# 추가
semanage port -a -t PORt_LABEL -p PROTOCOL NUM
ex)semanage port -a -t PORt_LABEL -p tcp 443
# 수정
semanage port -m -t PORt_LABEL -p PROTOCOL NUM
# 제거
semanage port -d -t PORt_LABEL -p PROTOCOL NUM


firewall-cmd --permanent --zone=public --add-port=443/tcp

firewall-cmd --get-default-zone
firewall-cmd --set-default-zone public
