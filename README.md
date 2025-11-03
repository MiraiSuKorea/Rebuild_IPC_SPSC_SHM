# Rebuild_IPC_SPSC_SHM

**1. Structure [100%] OFI logic added
**<img width="3845" height="3157" alt="Untitled diagram-2025-11-02-062227" src="https://github.com/user-attachments/assets/f507413d-c68b-445f-b253-b6ea1eccf29b" />

**2. FSM logic
**<img width="3665" height="5658" alt="Untitled diagram-2025-11-02-064258" src="https://github.com/user-attachments/assets/14a4166c-7006-439d-b580-502d7cce4617" />
<img width="3524" height="691" alt="Untitled diagram-2025-11-02-063911" src="https://github.com/user-attachments/assets/c8dc2d3b-b216-49d7-afa3-7759cf1d4c49" />

**3. Bitget demo trading [100% Fully working / cpp.map, IPC, shm_ring working perfect.]
**<img width="1068" height="1351" alt="image" src="https://github.com/user-attachments/assets/e0cbe5f4-3fec-40dd-97dc-dfd13530662a" />

**4. Programs [100%]
<img width="1818" height="915" alt="image" src="https://github.com/user-attachments/assets/9a261c79-7a09-477f-bc8c-23c67610509c" />
<img width="1053" height="449" alt="{CAC75DBD-2CF5-4319-950A-DB66B0337B71}" src="https://github.com/user-attachments/assets/9f33b04d-c225-45b9-9e3a-0550e1294d12" />

5. Data center [90%] local_ts is faster than server ts(impossible)
- Parquet data creator using same ring buffer
- Record Server timestamp & Local timestamp [For comparing latency & Real world simulation]
<img width="750" height="245" alt="{7C5A3834-5D9E-45D9-BB3F-61C6AB62155F}" src="https://github.com/user-attachments/assets/430f4c11-62f5-4efe-a447-a1a2478b77db" />
<img width="586" height="535" alt="{5D52492C-DB1E-4C8B-BC3F-A0FFDB60591C}" src="https://github.com/user-attachments/assets/2cdfd342-0135-4624-a318-9a74329619ed" />
<img width="579" height="607" alt="{A26BCBAD-AC14-48A1-A818-D2EA4D5918E4}" src="https://github.com/user-attachments/assets/4b7f7d7e-398e-4841-926b-5303e72b7816" />

<img width="544" height="427" alt="{B6F20C6A-314A-4D2A-B4B5-3FD0A4EC97CA}" src="https://github.com/user-attachments/assets/b470b395-761c-4ab9-b3a7-a315e7921106" />





7. Date Feeder [80%]

８. Signal [100%]
<img width="1012" height="570" alt="image" src="https://github.com/user-attachments/assets/3db73e45-cbb4-4e7e-94e6-847f300b60e7" />


9. Optimization []


10. Backtesting []


















================================================Tips===================================================
1. DPDK는 최후의 보루로
2. 일단 컨테이너들 네트워크 모드를 브릿지가 아닌 호스트로 연결하기
  
1. 도커 네트워킹 최적화 (Docker Networking Optimization) 
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


핵심은 **ENA(Elastic Network Adapter)**라는 AWS의 고성능 네트워크 인터페이스와 DPDK를 함께 사용하는 것입니다.

1. 어떻게 작동하나요?
ENA (Elastic Network Adapter): AWS의 '향상된 네트워킹(Enhanced Networking)'을 지원하는 EC2 인스턴스에 탑재된 가상 네트워크 카드입니다.

ENA PMD (Poll Mode Driver): DPDK는 이 ENA 하드웨어를 직접 제어할 수 있는 전용 드라이버(PMD)를 포함하고 있습니다.

커널 우회 (Kernel-Bypass): 이 드라이버를 사용하면, DPDK 애플리케이션(Cython 코드)이 리눅스 커널을 거치지 않고 ENA 네트워크 카드의 메모리(Ring Buffer)에 직접 접근하여 패킷을 읽고 쓸 수 있습니다.

결과적으로, Colocation 서버에서 DPDK를 쓰는 것과 동일한 원리(커널 우회)를 AWS EC2 인스턴스 상에서도 구현하여 패킷 처리 지연 시간을 극단적으로 줄일 수 있습니다.

2. AWS의 추가 최적화: ENA Express
최근에는 ENA를 한 단계 더 발전시킨 ENA Express라는 기능이 나왔습니다.

이는 AWS의 독자적인 **SRD(Scalable Reliable Datagram)**라는 프로토콜을 사용합니다.

DPDK 애플리케이션이 ENA Express를 활성화하면, TCP/IP 스택을 완전히 우회할 뿐만 아니라, AWS 백본망 내에서 패킷 순서 보장, 혼잡 제어 등을 하드웨어 레벨에서 처리하여 지연 시간을 더 안정적이고 낮게 유지해 줍니다.

HFT와 같이 지연 시간에 매우 민감한 워크로드에 특히 유리합니다.

3. 고려사항 (난이도)
인스턴스 유형: 모든 EC2 인스턴스가 지원하는 것은 아닙니다. c5n, m5n, r5n 등 'n'이 붙은 네트워크 최적화 인스턴스처럼 ENA와 향상된 네트워킹을 지원하는 인스턴스를 사용해야 합니다.

설정 복잡도: 단순히 apt-get install dpdk로 끝나지 않습니다.

특정 버전의 DPDK 소스를 컴파일해야 할 수 있습니다.

vfio-pci 같은 드라이버를 커널에 로드해야 합니다.

ENA 네트워크 인터페이스를 커널에서 분리(unbind)하여 DPDK에 바인딩하는 과정이 필요합니다.

HugePages 설정 등 시스템 튜닝이 필수입니다.


https://www.youtube.com/watch?v=DsNEtIS_q_E



2025.10.29
Orderbook std.map 매핑과정에서
30000.96 -> 이거 자체를 키값으로 만드니 변환과정에서 에러가 뜸
ex) 30000.95999999 -> 결국 업뎃안되고 Book_depth에 남아버림 -> OBI값 에러
적용해결책 / 헬퍼추
cdef int64_t price_str_to_int64(str price_str):

