# Rebuild_IPC_SPSC_SHM

<img width="801" height="641" alt="image" src="https://github.com/user-attachments/assets/768caa0f-9ec5-41ac-8f83-fcd7e7b895ea" />

<img width="1012" height="570" alt="image" src="https://github.com/user-attachments/assets/3db73e45-cbb4-4e7e-94e6-847f300b60e7" />


1. DPDK는 최후의 보루로
2. 일단 컨테이너들 네트워크 모드를 브릿지가 아닌 호스트로 연결하기
   

1. 도커 네트워킹 최적화 (Docker Networking Optimization)
 
가장 기본적이면서 효과적인 방법은 도커의 네트워크 오버헤드를 제거하는 것입니다.
 
Host 네트워크 모드 사용
 
적용: 컨테이너 실행 시 --network host 옵션을 사용합니다.
효과: 컨테이너가 호스트의 네트워크 스택과 IP 주소를 직접 공유함으로써, Bridge 모드에서 발생하는 NAT(주소 변환) 오버헤드와 가상 브리지 레이어를 완전히 제거하여 레이턴시를 최소화합니다.
 
2. 커널 및 TCP 스택 튜닝 (Kernel & TCP Tuning)
 
도커 컨테이너가 사용하는 리눅스 호스트의 커널 파라미터를 조정하여 TCP 성능을 개선합니다.
 
A. TCP Fast Open (TFO) 활성화
 
TCP Fast Open은 기존의 3-way handshake를 건너뛰고 데이터 전송을 시작하게 해, 연결 설정 시의 레이턴시를 줄입니다.
설정: net.ipv4.tcp_fastopen 값을 3으로 설정합니다.
 
B. Nagle 알고리즘 비활성화 (TCP NoDelay)
 
WebSocket은 실시간성이 중요하므로, 작은 패킷이라도 지연 없이 즉시 전송해야 합니다. Nagle 알고리즘은 네트워크 효율성을 위해 작은 패킷을 모아 전송하는데, 이는 레이턴시를 증가시킵니다.
적용: 애플리케이션 코드 (또는 서버 설정)에서 TCP NoDelay 옵션을 활성화하여 Nagle 알고리즘을 비활성화합니다.
 
C. 네트워크 버퍼 크기 조정
 
TCP/IP 버퍼 크기를 늘려 대용량 트래픽 처리 능력을 향상하고, 패킷 손실로 인한 재전송 가능성을 줄입니다.
설정: /etc/sysctl.conf에서 다음 파라미터들을 조정합니다.
net.core.rmem_max (수신 최대 버퍼)
net.core.wmem_max (송신 최대 버퍼)
net.ipv4.tcp_rmem, net.ipv4.tcp_wmem (TCP 수신/송신 메모리 한도)
 
D. Interrupt Coalescing 및 CPU Affinity
 
네트워크 인터럽트(IRQ) 처리의 효율성을 높입니다.
IRQ Affinity: 네트워크 인터럽트 처리를 특정 CPU 코어에 할당하여 캐시 미스(Cache Miss)를 줄이고 처리 효율을 높입니다.
Interrupt Coalescing: 패킷이 도착할 때마다 인터럽트를 발생시키는 대신, 일정 시간이나 패킷 수만큼 모아서 처리하여 CPU 사용률과 오버헤드를 줄입니다. (이 설정은 일반적으로 NIC 드라이버 수준에서 조정됩니다.)
 
3. 고급 Userspace 네트워킹 (Low Latency for Extreme Performance)
 
최소한의 레이턴시가 필요한 경우, 커널 네트워킹 스택의 오버헤드 자체를 완전히 우회하는 기술을 고려합니다.
 
A. DPDK (Data Plane Development Kit)
 
DPDK는 가장 낮은 레이턴시를 달성하는 방법 중 하나입니다.
원리: NIC(네트워크 카드)의 제어권을 커널에서 DPDK 드라이버로 옮겨와 커널 네트워킹 스택을 완전히 우회합니다. 패킷 처리를 Userspace에서 전용 CPU 코어(Polling Mode)를 사용하여 직접 처리하므로 인터럽트 오버헤드가 없습니다.
단점: 구현 복잡성이 매우 높고, 특정 NIC 하드웨어 지원이 필요하며, 컨테이너 내부에 DPDK를 지원하는 애플리케이션을 구동해야 합니다.
 
B. XDP (eXpress Data Path)
 
DPDK보다는 덜 극단적이지만, 커널 수준에서 고성능을 제공합니다.
원리: 패킷이 리눅스 네트워크 스택에 진입하는 가장 초기 단계에 eBPF 프로그램을 로드하여 빠른 처리를 가능하게 합니다.
효과: 패킷을 커널 스택으로 올리지 않고 드롭(Drop)하거나 다른 곳으로 리디렉션하는 작업을 초고속으로 처리할 수 있어, DDOS 방어, 로드 밸런싱 등에서 유용하며 네트워크 부하를 줄여줍니다.
 
4. 기타 애플리케이션 및 환경 설정
 
로드 밸런서/프록시 튜닝: NGINX, HAProxy 등을 사용할 경우, WebSocket은 장시간 연결이 유지되므로 연결 타임아웃 설정을 충분히 길게 지정해야 합니다. (예: Keepalive Timeout 설정)
CPU 리소스 제한 해제: 도커 컨테이너에 CPU 할당 제한(--cpus)이 너무 낮게 설정되어 있지 않은지 확인합니다. 네트워크 IO가 많은 경우 CPU 병목 현상이 레이턴시로 이어질 수 있습니다.
NUMA-Awareness: 서버가 NUMA 아키텍처를 사용하는 경우, 애플리케이션 프로세스, 네트워크 카드(NIC), 그리고 해당 메모리를 동일한 NUMA 노드에 할당하여 메모리 접근 레이턴시를 최소화합니다.
