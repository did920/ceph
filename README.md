# ceph
## ceph 도입을 위한 SRS  
클러스터 파일 시스템 개발 및 구축  
작업 기간 : 6개월, 혼자서 진행
### 개요 (Introduction)  
대용량 저장소의 자원사용을 최적화하기 위하여 클러스터 파일 시스템을 개발 및 구축하는 프로젝트이다.

### 요약 (Summary)  
> 확장성, 공유성, 장애대비, 자동복구능력이 가능한 파일시스템을 구축하기 위해 클러스터 파일 시스템중 하나인 Ceph를 개발 및 구축한다.  
> 확장성 : On-promise 스토리지, 클라우드 스토리지를 병합하여 사용할 수 있도록 사용 파일 스토리지, 오브젝트 스토리지 시스템을 구축한다.  
> 공유성 : 여러 Client가 하나의 스토리지에 접근하여 처리가능하도록 공유기능을 설정한다.  
> 장애대비 : 여러 스토리지가 묶인 통합 스토리지에 replica set을 2~3개를 설정하여 하나의 스토리지 서버가 down이 되도 사용 가능토록 한다.  
> 자동복구능력 : down된 서버를 자동 복구할 수 있도록 하드웨어 및 구성요소 설정을 한다.  


### 배경 (Background)  
> 무하유의 서비스가 계속 개발되고 기존 서비스의 규모가 커짐에 따라 다양한 데이터 및 대용량 데이터를 쉽게 접근하고 관리해야 하는 기능이 필요가 커졌다.  
> 백업, 복원, 부하 분산을 위해 대용량 데이터의 replica set을 많이 두는 것은 한계가 있으며, 대용량 데이터를 읽는 속도가 표절검사 서비스 성능의 병목이 되고 있다.  

### 목표 (Goals)  
> 대용량 데이터 읽기 성능 강화  
> 면접 동영상(몬스터)  
> 블랙홀 분석서버(카피킬러)  
> 스토리지 공유화(자원활용도↑)  
> 여러 client가 하나의 데이터를 공유가능토록 함(블랙홀 분석서버, 카피킬러)  
> 장애대응 능력 최적화  
> replica set에 따른 성능 최적화, 장애복구 시간에 대한 테스트 및 검증  

### 목표가 아닌  것 (Non-Goals)  
> 실시간 색인 서버그룹은 대상에서 제외한다. (Sqlite 특성상, read-only가 아니면 공유가 힘듦)  
> 파일서버, 백업서버들은 대상에서 제외한다. (백업과 replica 는 다른 개념, 파일서버는 local이 훨씬 안정적)  

### 이외 고려 사항들 (Other Considerations)  
> XEN 스토리지 호환성  
> Ceph 구성요소(mon, osd, mds, rgw, mgr)의 최적화  
> 모니터링 시스템 구축(prometheus + grafana)  

### 요구사항 상세 기술 (Requirement Specifications)  
서버 스펙
* MON
** CPU 2core 이상, MEM 64GB 이상, DISK 60GB 이상
* OSD
** CPU 디스크당 1~2 core, MEM 128GB 이상(디스크당 4~8GB), DISK 1TB 이상, OS와 OSD용 디스크는 분리
* MDS
** CPU 2core 이상 고클럭, MEM 2GB 이상, 
* RGW
** CPU 2core 이상, MEM 8GB 이상
* MGR
** mon과 같이

### 구성도
![image](https://user-images.githubusercontent.com/22390151/226381688-262db516-e5f1-475f-bcb9-d746bd5da9ab.png)




### 사용 기술 스택
> k8s, PROMETHEUS+GRAFANA
> 총 디스크는 48개, Replication set은 2으로 설정
> MDS는 2active, 1standby 구성
> OSD pool은 크게 한 개로 구성하여 Object, File 모두 동일 스토리지를 바라보도록 구성

### 설계에서 고려할 부분 (Design Considerations)  
> 네트워크 망을 10GB, 1GB, 2개 망으로 구축하여 데이터 전송용, health check용으로 나눈다.
> 모든 디스크에 Raid 구성은 제외한다
> OS kernel 버전(3.10.0-1160.49.1.el7.x86_64 ), Ceph 버전(Octopus 15.2.20 stable)은 같아야한다.
> 스토리지 접근은 쓰기용, 읽기용으로 나눠서 관리한다.
> 다른 인프라 환경과의 호환성

### 설계 상세 기술 (Design Specifications)
> 데이터 전송용 IP대역 10.10.100.X, health check용 IP대역 10.10.200.x 로 구성
> 다른 인프라 환경과의 연결은 VPN을 통해서 진행
![image](https://user-images.githubusercontent.com/22390151/226382419-36abb91e-41bc-4c86-9e22-55e92b770b71.png)


### 개발 가이드라인 (Development Guidelines)
> 스토리지를 실제 사용하면서 개선 사항을 발견하고 실시간 대응으로 개발해 나감

### 개발에서 고려할 부분 (Development Considerations)
> 기존 스토리지와 비교했을 때 성능 향상이 있어야만 반영(실제 성능, 안정성, 가성비 등)
