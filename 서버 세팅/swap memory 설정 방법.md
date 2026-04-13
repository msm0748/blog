# 2GB 크기의 스왑 파일 생성

```bash
sudo fallocate -l 2G /swapfile
```

이 명령어는 서버의 **가상 메모리(Swap)를 만들기 위해 2GB 크기의 빈 파일을 생성하는 명령어**입니다.

쉽게 비유하자면, 서버의 RAM(실제 메모리)이 부족할 때를 대비해 **하드디스크의 일부를 메모리처럼 쓸 수 있도록 '비상용 공간'을 미리 예약하는 작업**

---

### 🛠️ 남은 단계 이어서 하기

현재 `/swapfile`은 생성만 된 상태입니다. 아래 3개 명령어를 차례대로 입력해 보세요.

**1. 권한 설정 (보안)**

```bash
sudo chmod 600 /swapfile
```

**2. 스왑 포맷팅 (파일을 스왑 구역으로 변환)**

```bash
sudo mkswap /swapfile
```

**3. 스왑 활성화 (드디어 적용!)**

```bash
sudo swapon /swapfile
```

---

### ✅ 이제 다시 확인해 보세요

위의 3단계를 마치고 다시 확인 명령어를 치면 아래와 같이 결과가 나올 거예요.

```bash
sudo swapon --show
```

**예상 결과:**`NAME TYPE SIZE USED PRIO/swapfile file 2G 0B -2`

그리고 전체 메모리 상태를 보려면 이 명령어도 좋습니다.

```bash
free -h
```

`Swap:` 항목에 **2.0Gi**라고 표시된다면 성공입니다!

---

### ⚠️ 마지막 필수 단계 (재부팅 대비)

지금 설정한 건 서버를 껐다 켜면 초기화됩니다. **영구적으로 적용**하려면 아래 파일을 수정해야 합니다.

1. 파일 열기: `sudo vim /etc/fstab`
2. 맨 밑에 이 한 줄 추가: `/swapfile swap swap defaults 0 0`