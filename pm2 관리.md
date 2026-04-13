# ecosystem.config.cjs 파일 변경 사항 있을 때

## 1. 세팅값

```javascript
module.exports = {
  apps: [
    {
      /* 1. 기본 정보 설정 */
      name: 'koharu-market', // PM2 리스트에서 보일 앱의 이름입니다.
      cwd: '/home/msm0748/koharu-market', // 앱이 실행될 작업 디렉토리(폴더) 경로입니다.
      script: 'dist/src/main.js', // 실제로 실행할 자바스크립트 파일의 경로입니다.

      /* 2. 로그(기록) 설정 */
      // 모든 로그 줄 앞에 날짜와 시간을 붙여줍니다. (예: 2026-04-11 13:00:05 +09:00)
      log_date_format: 'YYYY-MM-DD HH:mm:ss Z',
      // console.error() 등으로 출력되는 에러 메시지를 저장할 파일 경로입니다.
      error_file: 'logs/error.log',
      // console.log() 등으로 출력되는 일반 메시지를 저장할 파일 경로입니다.
      out_file: 'logs/out.log',

      /* 3. 메모리 및 엔진 최적화 (가장 중요한 부분!) */
      // Node.js 엔진 자체에 전달할 옵션입니다.
      // 앱이 사용할 수 있는 최대 '힙 메모리(창고)' 크기를 512MB로 강제 지정합니다.
      node_args: '--max-old-space-size=512',

      // PM2가 밖에서 감시하다가, 앱의 전체 메모리 사용량이 512MB를 넘어가면
      // 서버 전체가 뻗지 않도록 앱을 자동으로 껐다 켜줍니다. (안전장치)
      max_memory_restart: '512M',

      /* 4. 실행 및 재시작 전략 */
      instances: 1, // 앱을 몇 개 띄울지 정합니다. (1개면 충분합니다.)
      exec_mode: 'fork', // 실행 모드입니다. 단일 인스턴스일 때 주로 'fork'를 씁니다.
      autorestart: true, // 앱이 에러로 죽었을 때 PM2가 자동으로 다시 살릴지 여부입니다.
      max_restarts: 15, // 앱이 계속 죽을 경우, 최대 15번까지만 다시 살리라는 뜻입니다. (무한 반복 방지)
      min_uptime: '10s', // 앱이 최소 10초는 살아있어야 '정상 실행'으로 간주합니다.

      /* 5. 환경 변수 설정 */
      env: {
        NODE_ENV: 'production', // 앱 내부에서 "아, 지금은 실제 서비스 중이구나"라고 인식하게 합니다.
        PORT: '3000', // 앱이 사용할 네트워크 포트 번호입니다.
      },
    },
  ],
};
```

## 2. 설정 적용하기 (중요)

파일을 저장한 후, PM2가 이 설정을 완전히 새로 읽어오도록 아래 명령어를 순서대로 실행해 주세요.

```bash
# 1. 기존에 돌고 있던 프로세스를 완전히 삭제
pm2 delete koharu-market

# 2. 새 설정 파일로 실행
pm2 start ecosystem.config.cjs

# 3. 서버 재부팅 시에도 이 설정으로 켜지도록 저장
pm2 save
```

# 로그 관리

## 1. 모듈 설치

```bash
pm2 install pm2-logrotate
```

## 2. 개별 설정 적용

```bash
# 1. 용량 제한: 로그 파일 하나당 10MB
pm2 set pm2-logrotate:max_size 10M

# 2. 보관 개수: 최대 10개만 남김
pm2 set pm2-logrotate:retain 10

# 3. 압축: 오래된 로그는 압축해서 저장 (용량 절약 핵심!)
pm2 set pm2-logrotate:compress true

# 4. 강제 주기: 매일 자정(00:00)에 무조건 파일을 새로 나눔
pm2 set pm2-logrotate:rotateInterval '0 0 * * *'
```

## 3. 설정 확인하기

설정을 다 하셨다면 아래 명령어로 값이 잘 들어갔는지 최종 확인해 보세요.

```bash
pm2 conf pm2-logrotate
```

# 로그 확인

## 1. 실시간 로그 확인 (가장 많이 씀)

앱이 현재 어떻게 돌아가고 있는지, 요청이 들어오는지 실시간으로 지켜볼 때 사용합니다.

- **모든 로그 실시간 보기:**
    
    ```bash
    pm2 logs koharu-market
    ```
    
- **최신 로그 50줄 출력 후 실시간 대기:** (숫자는 조절 가능)
    
    ```bash
    pm2 logs koharu-market --lines 50
    ```
    
- **에러 로그만 골라보기:** (빨간 글씨 위주로 보고 싶을 때)
    
    ```bash
    pm2 logs koharu-market --err
    ```
    

---

## 2. 과거 로그 파일 확인 (파일로 저장된 것)

아까 `ecosystem.config.cjs`에서 로그 경로를 지정했기 때문에, 이제 파일로 직접 열어볼 수 있습니다.

- **프로젝트 폴더 내 로그 확인:**
```bash
# 일반 출력 로그 마지막 100줄 보기
tail -n 100 logs/out.log

# 에러 로그 마지막 100줄 보기
tail -n 100 logs/error.log
```

- **실시간으로 파일 감시하기 (`tail -f`):**
 ```bash
 tail -f logs/out.log
 ```

---

## 3. 로그 관리 및 초기화

- **현재까지 쌓인 로그 싹 비우기:** (디스크 용량 확보나 새 기분으로 시작할 때)
```bash
pm2 flush
```
- **로그 로테이트 설정 확인:** (아까 설정한 값들이 잘 있나 볼 때)
```bash
pm2 conf pm2-logrotate
```

---

## 4. 시각적인 모니터링 (강력 추천)

글자로만 보는 게 아니라 전체적인 시스템 상태와 로그를 한눈에 볼 때 가장 좋습니다.

```bash
pm2 monit
```

- **왼쪽 상단:** 프로세스 리스트 (상태 확인)
- **오른쪽 상단:** 실시간 로그 (위아래 화살표로 스크롤 가능)
- **하단:** 메모리(Heap) 및 CPU 점유율 실시간 그래프