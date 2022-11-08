---
title: "JVM: error=12, Cannot allocate memory or error=12"
toc: true
toc_sticky: true
categories:

- develop

tags:

- java
- Spring Boot

---

# JVM: error=12, Cannot allocate memory

업무를 진행하면서 모종의 이유로 인해 인스턴스가 죽는 현상이 발생하였다. `hikari pool size`에 대한 이슈로 스레드의 개수에 따른 `JPA database connection pool size`의 개수
때문에 발생하였는데 해당 부분에 대해 조치 후 프로세스를 기동하여도 기동이 되지 않는 증상이 다시 발생하였다.

## 증상

![img.png]({{site.url}}/assets/images/develop/jvm/allocate_memory/jvm_not_enough_space.png)

## 원인

`error='Not enough space'`에 주목하자

이 오류는 JAVA가 프로세스를 포킹하는 과정에서 발생하는 이슈로 시스템의 메모리 또는 스왑이 부족한 경우 발생한다.

실제로 서버의 메모리 사용량을 확인하였을때 사용 가능한 메모리가 필요한 메모리보다 적거나 사용중인 메모리가 가득 차 있는 것을 확인 가능하다.

> 샘플 이미지를 추가하고 싶으나 발생 시점으로부터 시간이 지난시점에 문서를 작성하여 실제 메모리 이미지는 확보를 하지 못하였으며, 임의로 발생 시킬 수 있으나 귀찮다...

## 해결

이 증상의 해결 방법은 아주 간단하며 아래와 같은 해결 방법이 있을 것 같다.

1. 인스턴스의 메모리를 증가시키는 방법
  - 이 방법은 클라우드 또는 VM 환경에서 사용 가능하다.
2. 스왑 메모리를 증가 시키는 방법
  - 이 방법은 모든 환경에서 사용 가능하다.
3. Java heap 메모리 줄이기
4. Java thread 줄이기

등....

나는 이 두 방법중 2번의 스왑 메모리 증가하는 방법으로 해결하였으며 이 방법으로 포스팅 하겠다.

### 스왑 메모리 할당

```shell
# swap memory 확인
$ free -h
 
# swap file 생성
$ touch /var/spool/swap/swapfile

# swap에 2GB의 메모리를 할당
$ dd if=/dev/zero of=/var/spool/swap/swapfile count=2048000 bs=1024

# swap에 권한 설정
$ chmod 600 /var/spool/swap/swapfile

# 파일 포멧을 swap으로 변환하고 swap file로 등록
$ mkswap /var/spool/swap/swapfile
$ swapon /var/spool/swap/swapfile

# 파일 시스템 테이블 등록
$ vim /etc/fstab 

# 아래 내용 등록후 저장
/var/spool/swap/swapfile none swap defaults 0 0

# swap 메모리 확인
$ free -h
```

최종적으로 메모리를 화인하면 swap 메모리가 추가된 것을 확인할 수 있다.

# Reference

- https://confluence.atlassian.com/stashkb/forking-jvm-error-12-cannot-allocate-memory-or-error-12-not-enough-space-302809889.html
- https://velog.io/@adam2/JVM-Cannot-allocate-memory-%EC%97%90%EB%9F%AC