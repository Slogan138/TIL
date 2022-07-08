# Why am i writing it?
OpenStack Octavia 에서 Health Manager Port 를 어떻게 설정하는지 작성한다.

# What is Octavia?
Octavia 란 OpenStack 에서 Load Balancer 기능을 위해 제공하는 서비스 모듈이다.

# Problem Occurred!
OpenStack 에서 Load Balancer 모니터링 기능을 위해 조사하던 도중 [Load Balancer Statistics API](https://docs.openstack.org/api-ref/load-balancer/v2/#get-load-balancer-statistics)  를 Octavia 에서 제공해 주는 것을 확인했다.   
다만 구축되어 있는 OpenStack 서버에서는 생성된 Load Balancer 에 트래픽을 발생시켰음에도 불구하고 해당 API 값이 모두 0으로 오고 있었다.   
```shell
root@con01:~# openstack loadbalancer stats show b56a7ec9-967a-410b-a17d-ad0f814f5a88
+--------------------+-------+
| Field              | Value |
+--------------------+-------+
| active_connections | 0     |
| bytes_in           | 0     |
| bytes_out          | 0     |
| request_errors     | 0     |
| total_connections  | 0     |
+--------------------+-------+
```

# What is the cause?
![LB_Operating_Status](/OpenStack/Octavia/image/LB_Operating_Status.png)   
Load Balancer 를 생성했을 때 기능이 정상적으로 동작 했지만 왜인지 운영 상태는 오프라인 상태였다.   
Statistics API 가 모두 0인 이유가 운영 상태와 연관이 있지 않을까 생각되었다.

# Octavia Installation
Octavia 를 설치하는 과정에서 Octavia Subnet 을 생성하고 External Network 와 연결한다.
```shell
$ openstack network create --share --provider-network-type vxlan octavia-net
$ openstack subnet create --network octavia-net --dns-nameserver 8.8.8.8 --gateway 20.0.0.1 --subnet-range 20.0.0.0/24 octavia-sub
$ openstack router add subnet external-router octavia-sub
```

이후 Controller Node 에서 NAT Network 로 Octavia Network IP 를 Destination IP 로 되어있는 Packet 전송 시, 해당 Packet 이 External Router 로 전송할 수 있도록 Controller 에 Routring Rule을 추가한다.
```shell
$ route add -net 20.0.0.0/24 gw 192.168.0.225
$ printf '#!/bin/bash\nroute add -net 20.0.0.0/24 gw 192.168.0.225' > /etc/rc.local
$ chmod +x /etc/rc.local
```

이러한 과정을 거치지 않으면 Controller Node 에서 Octavia Network 가 통신하지 못하기 때문에 Load Balancer 생성 과정중에 Error 상태가 된다.

# How to Configure Health Manager Port
Octavia 설치 이후 octavia.conf 에 health_manager 섹션 설정들을 수정해야 한다.

## Pre Configuration
Octavia Network 와 통신을 하기 위한 Port 를 생성한다.
```shell
$ openstack port create octavia-hm-port-con01 --host con01 --network octavia-net
```

위 명령어를 통해 만들어진 IP 를 OpenVSwitch 명령어로 가상 Port 를 만든다.
```shell
MGMT_PORT_ID=$(openstack port show octavia-hm-port-con01 | awk '/ id / {print $4}')
MGMT_PORT_MAC=$(openstack port show octavia-hm-port-con01 | awk '/ mac_address / {print $4}')
HMIP=$(openstack port show octavia-hm-port-con01 | awk '/ fixed_ips / {print $4}' | cut -d "'" -f 2)

echo $MGMT_PORT_ID
echo $MGMT_PORT_MAC
echo $HMIP

ovs-vsctl -- --may-exist add-port br-int octavia-hm0 -- set Interface octavia-hm0 type=internal -- set Interface octavia-hm0 external-ids:iface-status=active -- set Interface octavia-hm0 external-ids:attached-mac=$MGMT_PORT_MAC -- set Interface octavia-hm0 external-ids:iface-id=$MGMT_PORT_ID
sudo ip link set dev octavia-hm0 address $MGMT_PORT_MAC
ifconfig octavia-hm0 $HMIP/24
```

각 Controller Node 마다 Port 를 만들어 위와 같은 과정을 반복한다.

## Configure Port
Pre Configure 과정에서 각 Controller Node 마다 생성된 Port IP를 octavia.conf 파일에 설정한다.
```text
[health_manager]
bind_port = 5555
bind_ip = 20.0.0.83
heartbeat_key = insecure
controller_ip_port_list = 20.0.0.191:5555,20.0.0.34:5555,20.0.0.83:5555
stats_update_threads = 5
health_update_threads = 5
```

bind_port 항목을 Port IP 로 설정한다.
controller_ip_port_list 항목은 각 Controller Node 마다 생성된 Port IP 에 5555 Port 로 하여 리스트 형식으로 작성한다.

# Conclusion
Health Manager 설정을 모두 마치고 Octavia 서비스를 재시작한다면 생성 이후 운영 상태가 오프라인으로 머물러 있던것이 온라인으로 변경된다.   
또한 생성된 Load Balancer 에 트래픽을 발생시키고 Load Balancer Statistic API 를 호출하면 정상적으로 증적된 데이터를 가져올 수 있다.
```shell
root@con01:~# openstack loadbalancer stats show c3c04e34-4ab2-47c2-a236-2733b7bf5b2e
+--------------------+---------+
| Field              | Value   |
+--------------------+---------+
| active_connections | 0       |
| bytes_in           | 27887   |
| bytes_out          | 1823598 |
| request_errors     | 0       |
| total_connections  | 353     |
+--------------------+---------+
```

### Reference.
- https://ssup2.github.io/record/OpenStack_Stein_%EC%84%A4%EC%B9%98_Kolla-Ansible_Ubuntu_18.04_ODROID-H2_Cluster/#13-external-network-octavia-network-%EC%83%9D%EC%84%B1