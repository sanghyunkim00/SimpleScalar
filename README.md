# SimpleScalar

# run.sh
You can run SimpleScalar as follows.
```
chmod +x run.sh # if run.sh isn't executable.
# ./run.sh <num_threads> <config_name> [benchmark_name]
./run.sh 1 example.cfg bzip2
```
NOTE: You must create the config file yourself.

# SimpleScalar Tips

## WSL 설치
- 본 프로젝트는 Linux 환경에서 컴파일됩니다. 개발 및 실행을 위해 Linux 사용을 권장합니다.  
- Windows 사용자는 WSL(Windows Subsystem for Linux) 설치를 권장하지만, 필수는 아닙니다. VirtualBox/VMware 위의 Linux나 네이티브 Linux를 사용해도 됩니다.  
- WSL 설치 방법은 아래 문서를 참고하세요.  
	- https://learn.microsoft.com/ko-kr/windows/wsl/install


## Multi-Program Input
- SMT 실행을 위해서는 여러 개의 프로그램을 입력으로 받을 수 있도록 수정해야 합니다.  
- 각 입력 프로그램은 구분자 “--”로 나눈다는 가정하에 **run.sh**를 작성했습니다. 기본적으로 “--” 사용을 권장하지만, 다른 방식으로 구현해도 무방합니다.  
- 참고: 프로그램 로드는 **main.c**에서 **sim-outorder.c**의 *sim_load_prog()* 을 호출하여 수행됩니다. SMT를 구현하려면 이 부분을 반드시 수정해야 합니다.


## 구현 TIPS
### Virtual Address
- SimpleScalar의 모든 캐시(I-cache, D-cache, TLB, BTB 등)는 가상 주소로 동작합니다.  
	- 각 스레드는 독립적인 *struct mem_t*가 필요합니다.  
- 따라서 각 리소스에 TID를 추가해, 해당 엔트리가 어느 스레드의 것인지 반드시 구분해야 합니다.  
- 구현 위치 참고:
	- I-Cache, D-Cache, TLB: **cache.c**
	- Branch Predictor(BTB, RAS): **bpred.c**
### Register File
- SimpleScalar의 리네이밍 로직은 ROB 기반과 유사하며, struct RUU_station이 ROB 역할을 수행합니다.  
	- RUU는 reservation station과 ROB를 합친 SimpleScalar의 단위입니다.  
- 따라서 ROB 엔트리는 스레드별로 반드시 구분해야 합니다. 구현 방식은 정적 또는 동적으로 선택할 수 있습니다.  
- 또한 각 스레드는 독립적인 architectural register file이 필요하므로, *struct regs_t*를 스레드별로 할당해야 합니다.  
- 참고: 레지스터 파일 관리를 위해 아래 변수/함수가 사용됩니다.
	- use_spec_cv
	- create_vector
	- spec_create_vector
	- create_vector_rt
	- spec_create_vector_rt
### sim_main()
- simpulation logic은 sim_outorder.c의 sim_main()에 구현되어있습니다.
- 해당 함수를 참고하여 SMT를 구현하면 됩니다.
