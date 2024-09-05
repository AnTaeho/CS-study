## EC2 프리티어의 메모리 부족 현상을 Swap 가상 메모리 설정으로 해결

### 문제 상황

- AWS EC2 프리티어 제품 사용중 메모리 부족으로 서버가 다운되는 문제 발생

### **EC2 인스턴스 스팩**

- 메모리 : 1GB
- 스토리지 : 20GB

### 해결 방법

- Swap 파일을 생성해서 RAM의 공간이 부족할 때 단기적으로 스왑 공간을 대체로 사용하도록 설정
- 권장되는 스왑 메모리 크기에 따라서 1GB의 2배인 2GB로 설정
- AWS 에서 제공하는 공식문서를 참고했습니다. [AWS 공식 문서](https://repost.aws/ko/knowledge-center/ec2-memory-swap-file)
```java
1. dd 명령을 사용하여 루트 파일 시스템에 스왑 파일을 생성합니다.
$ sudo dd if=/dev/zero of=/swapfile bs=128M count=16

2. 스왑 파일의 읽기 및 쓰기 권한을 업데이트합니다.
$ sudo chmod 600 /swapfile

3. Linux 스왑 영역을 설정합니다.
$ sudo mkswap /swapfile

4. 스왑 공간에 스왑 파일을 추가하여 스왑 파일을 즉시 사용할 수 있도록 합니다.
$ sudo swapon /swapfile

5. 절차가 성공적으로 완료되었는지 확인합니다.
$ sudo swapon -s

6. 편집기에서 파일을 엽니다.
$ sudo vi /etc/fstab

7. 파일 끝에 다음 새 줄을 추가하고 파일을 저장한 다음 종료합니다.
/swapfile swap swap defaults 0 0
```

### 결과

- 스왑 메모리 공간을 설정해서 약 2GB의 임시 메모리 공간을 확보했다.
  <img src="https://tech-blog-image.s3.ap-northeast-2.amazonaws.com/image/005b0101-ff13-44dd-9407-9a28a419f71f3c2.png" alt="이미지" style="max-width: 100%; height: auto; display: block; margin: 0 auto;"/>
