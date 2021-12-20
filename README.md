# AWS-Elemental-MediaLive

* * *

### # AWS MediaLive Fail-Over 동작 방식

- [1] AWS MediaLive & MediaStore Fail-Over 동작 방식
- [2] AWS MediaLive & MediaPackage Fail-Over 동작 방식


* * *

#### [1] AWS MediaLive & MediaStore Fail-Over 동작 방식 (상세)

- 1-1. AWS Elemental MediaLive & MediaStore 이중화 채널 
    - MediaStore 구성에서는 재생할 수 있는 Master Manifest 재생 주소가 파이프라인 별 1개씩 생성 (A와 B 2개)

![image](./images/Capture-1.png)

- 1-2. AWS Elemental MediaStore 구성에서의 Fail-Over 설정
    - Fail-Over 동작을 위해 MediaLive의 Redundant Manifest & Puase Output 설정 필요
        - Redundant Manifest : 각 파이프라인의 Sub Manifest를 중복으로 저장
        - Puase Output : 파이프라인으로의 입력 스트림이 중단되면 출력 스트림 또한 중단
    - Puase Output 설정은 [Input Loss Action]에서 가능하며 선택 가능한 설정 값은 2종류
        - PAUSE_OUTPUT : 이중화 채널(Standard)에서 사용되며 입력 스트림이 끊어지면 해당 파이프라인을 중단
        - EMIT_OUTPUT : 싱글 채널(Single)에서 사용되며 입력 스트림이 끊어져도 블랙 프레임을 생성하여 스트림을 전달

- 1-3. AWS Elemental MediaLive의 Redundant Manifest DISABLED(비활성화) 동작 방식
    - 채널의 파이프라인 A와 B는 각각의 파이프라인에 해당하는 Sub Manifest만 보유
        - A의 Master Manifest 주소로 재생하고 있을 때, A 파이프라인에 입력 신호가 중단되면 장애가 발생        
        - A의 Master Manifest 주소는 재생 불가, B의 Master Manifest 주소로 전환해 주는 방안이 필요
    
<br><br>

![image](./images/Capture-2.png)

- 1-5. AWS Elemental MediaLive의 Redundant Manifest ENABLE(활성화) 동작 방식
    - 채널 파이프라인 A와 B의 Master Manifest에서 각 파이프라인에 해당하는 두 곳의 Sub Manifest를 모두 보유
    

<br>

![image](./images/Capture-3.png)

-  1-6. 채널 파이프라인 A와 B가 두 곳의 Sub Manifest를 모두 보유하게 될 경우 동작 방식
    - A 파이프라인에 입력 신호가 중단되면 A Master Manifest에서 A의 Sub Manifest가 삭제
        - A Master Manifest 주소로 동영상을 재생하고 있다고 가정하에
        - A 파이프라인에 입력 신호가 중단되면 입력 신호가 입수되어 처리되고 있는 B의 Sub Manifest로 재생

<br>

![image](./images/Capture-4.png)

- 1-7. 해당 동작이 가능하기 위해서는 AWS Elemental MediaLive에서 다음과 같은 설정이 필요
    - [HLS OutputGroup] > [Manifest and Segments] > [Redundant Manifest] > ENABLED
    - [HLS OutputGroup] > [HLS settings] > [Input Loss Action] > PAUSE_OUTPUT


<br>

* * *

#### [2] AWS MediaLive & MediaPackage Fail-Over 동작 방식 (상세)

- 2-1. AWS Elemental MediaLive & MediaPackage 이중화 채널
    - MediaStore 구성에서는 재생할 수 있는 Master Manifest 재생 주소가 1개만 생성
    - MediaStore 구성과는 다르게 파이프라인의 Fail-Over 전환이 내부적으로 처리되기에 재생 주소는 1개만 생성

- 2-2. AWS Elemental MediaPackage 구성에서의 Fail-Over 설정
    - Fail-Over 동작을 위해 MediaLive의 Puase Output 설정 필요
        - Puase Output : 파이프라인으로의 입력 스트림이 끊기면 출력 스트림을 중단
    

![image](./images/Capture-5.png)

- MediaLive로의 A 파이프라인이 송출이 중단되면 PAUSE_OUTPUT 동작으로 MediaPackage에 더 이상 스트림을 보내지 않고 중단
   

<br><br>

![image](./images/Capture-6.png)

- MediaLive에서 A 파이프라인의 스트림이 입수되지 않음을 알게 된 MediaPackage는 활성 스트림을 B로 전환

<br>

![image](./images/Capture-7.png)

- 2-3. AWS Elemental MediaPackage를 미디어 스토리지로 사용한 구성 시 Fail-Over 동작 방식
    - 즉, 파이프라인 A의 송출이 끊어져도 MediaPackage에서 사용 가능한 활성 스트림을 자동으로 전환하는 방식
    - MediaPackage 내부적으로 Fail-Over 전환이 발생하기에 사용자는 엔드포인트 URL 전환에 대한 고려가 필요 없음

<br>

![image](./images/Capture-8.png)

- 2-4. 해당 동작이 가능하기 위해서는 AWS Elemental MediaLive에서 위와 같은 설정이 필요

    - [HLS OutputGroup] > [HLS settings] > [Input Loss Action] > PAUSE_OUTPUT

<br>

- 원활한 MediaLive의 Fail-Over를 위해 MediaSotre의 (d. Set up object lifecycle policy) 설정이 추가로 필요하다.
https://aws.amazon.com/ko/blogs/media/build-a-reliable-hls-live-channel-with-aws-elemental-mediastore/

* * *

 #### AWS MediaLive & MediaPackage Fail-Over 동작 방식 (참고 사이트)   

![image](./images/Capture-9.png)

- 실제 라이브 스트리밍 파이프라인이 MediaPackage에서 Fail-Over 되는 예제를 살펴볼 수 있다.
https://www.youtube.com/watch?time_continue=48&v=5eBkZInd8kQ&feature=emb_logo

<br><br>

![image](./images/Capture-10.png)

- 원활한 MediaPackage의 Fail-Over를 위해 MediaLive에서는 위와 같은 설정이 필요할 수 있다.
https://aws.amazon.com/ko/blogs/media/part1-how-to-set-up-a-resilient-end-to-end-live-workflow/

<br><br>
