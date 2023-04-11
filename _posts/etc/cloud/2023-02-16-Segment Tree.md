---
title: "[Container] Container Run-time 1"
excerpt: "What is Container?"

categories:
  - cloud
tags:
  - [container]

toc: true
toc_sticky: true

date: 2023-04-11
last_modified_at: 2023-04-11
---

 컨테이너를 다룰 때 많이 듣는 용어 중 하나는 "컨테이너 런타임" 입니다. "컨테이너 런타임" 의 의미는 표준에 따라 다를 수 있습니다. 

 포스팅에서는 컨테이너 런타임을 알아보기 전에 컨테이너란 무엇인가? 에 대해서 알아보겠습니다.

# Container

컨테이너란 무엇일까요? 흔히들 컨테이너는 VM 과 비교를 합니다. 가장 중요한 특징은 호스트와 분리된 별도의 프로세스라는 것이죠. 이를 가능케하는 것은 'namespace' 와 'cgroups' 입니다.

## namespaces

네임스페이스는 하나의 프로세스를 다른 프로세스와 분리시키는 역할을 합니다. 

- pid namespace
  모든 각자의 네임스페이스의 PID 는 1부터 시작합니다. 하나의 호스트 위에 올라간 여러 네임스페이스들도 모두 PID 1을 가지지만 서로 다른 프로세스입니다.

- networking namespace
  docker 에서 컨테이너를 사용할 때 port 번호를 지정합니다. 이때 지정하는 포트번호는 당연히 호스트와 별개의 port 입니다.

- mount namespace
  네임스페이스에서 mount, unmount 를 하더라도 호스트에 영향을 끼치지 않습니다.

위를 통해 결국 namespace 는 host 에서 프로세스, 네트워킹, 파일 시스템을 모두 분리시킬 수 있다는 것을 확인할 수 있습니다. 

```
$ sudo unshare --fork --pid --mount-proc bash
```

<img width="700" alt="image" src="https://user-images.githubusercontent.com/56664567/231110675-293aa4fd-13a8-48a0-84c1-c0b2bee1cb16.png">

unshare 는 namespace 를 분리하는 명령어입니다. 'ps' 를 통해 namespace 에서 실행되는 프로세스가 PID 1 부터 시작하는 것을 확인할 수 있습니다.

### nsenter

분명 격리되어있지만 다른 namespace 에 접근할 수 있는 방법은 존재합니다. 'nsenter' 명령어를 사용하면 다른 namespcae 에 접근해서 명령어를 실행시킬 수 있습니다. 이를 통해 컨테이너에 진입하는 'docker exec' 도 결국 'nsenter' 명령어를 사용하는 것임을 알 수 있습니다.

## cgroups

cgroups 는 한마디로 "resource limits" 이라고 할 수 있습니다. VM 을 사용할 때 가장 처음 cpu 의 성능이나 memory 의 크기, vloume 의 크기 등을 설정하는 것 처럼 컨테이너를 사용할 때도 리소스의 제한을 설정해야합니다. 이를 cgroups 를 통해 설정할 수 있습니다. 

```
$ sudo cgcreate -a bork -g memory:mycoolgrou
```

<img width="700" alt="image" src="https://user-images.githubusercontent.com/56664567/231111308-8aae53da-4e33-4b8a-b48b-f98ea2a20daf.png">

'cgcreate' 명령어로 cgroup 을 생성할 수 있습니다. 생성한 cgroup 내부를 살펴보면 두 파일이 존재하는 것을 확인할 수 있습니다.

- memory.kmem.limit_in_bytes : 이 파일은 실제 CPU, Memory 의 제한량을 설정할 수 있습니다. 
- memory.kmem.max_usage_in_bytes : 이 파일은 현재 해당 cgroup 에서의 CPU, Memory 사용량을 나타냅니다. 읽기 전용 파일로 모니터링 등에 활용할 수 있습니다.


## seccomp-bpf 

추가적으로 이 명령어는 프로세스에서 시스템 콜을 제한하는 기능을 가지고 있습니다.

Docker에서는 seccomp 프로파일링 기능을 사용하여 컨테이너 내에서 실행되는 애플리케이션에 대한 시스템 호출을 제한할 수 있습니다. seccomp 프로파일은 컨테이너가 시작될 때 Docker에 의해 로드되며 컨테이너 내부의 프로세스에 적용됩니다.

다른 예시로는 Chrome 브라우저가 seccomp-bpf를 사용하여 브라우저에서 실행되는 웹 페이지에서 시스템 호출을 제한합니다. 이를 통해 악성 코드가 브라우저에서 실행되는 운영 체제에 액세스하지 못하도록 합니다.

다음은 seccomp-bpf 를 설정하는 예시입니다.
아래와 같이 BPF 어셈블리 코드를 작성합니다.

```
BPF_MOV64_REG(BPF_REG_6, BPF_REG_1);
BPF_ALU64_IMM(BPF_ADD, BPF_REG_6, -1);
BPF_LD_IMM64(BPF_REG_2, 0x0);
BPF_JMP_REG(BPF_JGT, BPF_REG_2, BPF_REG_6, 1);
BPF_EXIT_INSN();
BPF_LD_ABS64(BPF_B, offsetof(struct seccomp_data, arch));
BPF_JMP_IMM(BPF_JNE, BPF_REG_0, AUDIT_ARCH_X86_64, 2);
BPF_LD_ABS64(BPF_W, offsetof(struct seccomp_data, nr));
BPF_JMP_IMM(BPF_JNE, BPF_REG_0, __NR_open, 0);
BPF_JMP_IMM(BPF_JEQ, BPF_REG_0, __NR_close, 1);
BPF_EXIT_INSN();
```

이후 필터 적용 명령어를 실행하면 해당 프로세스에서 시스템 콜을 제한시킬 수 있습니다.

```
sudo prlimit --pid <PID> --seccomp=filter ./<EXECUTABLE_FILE>
```

이 외에도 seccomp() 시스템 호출을 사용하여 프로세스 내부에서 seccomp-bpf 필터를 적용할 수 있습니다.

---

<img width="700" alt="image" src="https://user-images.githubusercontent.com/56664567/231113392-4f57bfc2-c1cf-4b5a-b762-4587e0e33649.png">

위 그링은 어디서 한 번쯤은 봤을 법한 docker Container 과 VM 의 차이이다. 
container 는 Host OS 를 공유하고 그 위에서 동작하는 하나의 프로세스라고 말할 수 있다. 여기서 핵심은 각 container 는 VM 처럼 철저하게 분리되어있다는 것이다. 따라서 OS 가 직접 올라가는 VM 보다 훨씬 적은 이미지만으로 실행될 수 있다. 

결국 container 는 namespace, cgroups 를 통해 host 및 다른 프로세스와 분리된 하나의 프로세스라고 말할 수 있다. 