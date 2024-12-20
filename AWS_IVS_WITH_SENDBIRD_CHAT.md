1. 요구 사항

호스트(Host): 스트림을 시작하고 종료하는 주체.

사용자(User): 스트림에 참여해 비디오를 시청하고 채팅에 참여.

Sendbird Chat: 실시간 채팅 관리.

AWS IVS: 라이브 스트리밍 관리.



2. AWS IVS JAVA SDK

설치

dependencies {
    implementation 'software.amazon.awssdk:ivs:2.20.28'
}

사용

import com.amazonaws.services.ivs.AmazonIVS;
import com.amazonaws.services.ivs.AmazonIVSClientBuilder;
import com.amazonaws.services.ivs.model.CreateChannelRequest;
import com.amazonaws.services.ivs.model.CreateChannelResult;

public class IvsService {

    private final AmazonIVS ivsClient;

    public IvsService() {
        this.ivsClient = AmazonIVSClientBuilder.standard().build();
    }

    public CreateChannelResult createChannel(String channelName) {
        CreateChannelRequest request = new CreateChannelRequest()
                .withName(channelName)
                .withLatencyMode("LOW") // 또는 "NORMAL" 설정 가능
                .withType("STANDARD"); // STANDARD 또는 BASIC 설정 가능
        return ivsClient.createChannel(request);
    }
}

 응답값

channelArn: 채널의 고유 식별자.

streamKey: RTMP 스트림 시작에 필요한 키.

playbackUrl: 사용자용 HLS 스트림 URL.



호스트 스트림 시작

호스트는 RTMP 스트림을 시작합니다. 이를 위해 클라이언트가 생성된 streamKey와 RTMP URL을 사용해야 합니다.

rtmp://<channel-ingest-server>.ivs.rocks/app/<streamKey>





스트림 상태 확인

호스트가 스트림을 시작한 이후, 스트림이 실제로 진행 중인지 확인하려면 GetStream API를 호출합니다.

import com.amazonaws.services.ivs.model.GetStreamRequest;
import com.amazonaws.services.ivs.model.GetStreamResult;

public boolean isStreamLive(String channelArn) {
    try {
        GetStreamRequest request = new GetStreamRequest().withChannelArn(channelArn);
        GetStreamResult result = ivsClient.getStream(request);
        return result.getStream() != null; // 스트림이 활성 상태인지 확인
    } catch (Exception e) {
        System.out.println("Stream is not live: " + e.getMessage());
        return false;
    }
}

스트림이 활성 상태이면 true, 그렇지 않으면 false.





스트림 종료

호스트가 스트림을 종료하면 StopStream API를 호출해 스트림을 중단할 수 있습니다.

import com.amazonaws.services.ivs.model.StopStreamRequest;

public void stopStream(String channelArn) {
    StopStreamRequest request = new StopStreamRequest().withChannelArn(channelArn);
    ivsClient.stopStream(request);
    System.out.println("Stream stopped for channel: " + channelArn);
}





3. Sendbird Chat 통합

AWS IVS의 채널과 연결된 Sendbird 채팅 채널을 생성합니다.

curl -X POST https://api-{application_id}.sendbird.com/v3/group_channels \
-H "Api-Token: {api_token}" \
-H "Content-Type: application/json" \
-d '{
  "name": "Live Stream Chat - {channelName}",
  "channel_url": "chat_{channelArn}",
  "is_distinct": true,
  "user_ids": ["host_id"]
}'

 채팅 참여 (사용자 추가)

curl -X PUT https://api-{application_id}.sendbird.com/v3/group_channels/{channel_url}/invite \
-H "Api-Token: {api_token}" \
-H "Content-Type: application/json" \
-d '{
  "user_ids": ["user1", "user2"]
}'





4. 전체 스트림 플로우

a. 호스트 스트림 시작:

AWS IVS에서 CreateChannel로 채널 생성.

streamKey 및 RTMP URL을 호스트에게 전달.

스트림 시작 후 GetStream으로 상태 확인.

b. 사용자 스트림 참여:

playbackUrl을 사용자에게 제공하여 스트림 시청 가능.

Sendbird Chat의 channel_url로 채팅 채널 초대.

c. 호스트 스트림 종료:

AWS IVS의 StopStream으로 스트림 종료.

Sendbird Chat에서 필요 시 채널 삭제.



5. AWS IVS 장점

초저지연(초 단위 이하) 스트리밍:

초저지연 모드(Low Latency Mode)를 통해 실시간 상호작용이 필요한 스트리밍 환경에 적합.



손쉬운 S3 버킷 연동:

IVS는 생성한 채널에 대해 녹화를 활성화하면 자동으로 S3 버킷에 녹화된 비디오 파일(HLS)을 업로드할 수 있습니다.

활용 사례:

라이브 스트리밍 종료 후 다시보기 기능 제공.



글로벌 네트워크 기반:

AWS의 글로벌 네트워크를 기반으로 안정적이고 빠른 스트리밍 환경 제공.

전 세계 사용자에게 동일한 품질로 콘텐츠 전달 가능.





6. AWS IVS 단점

1.커스터마이징 한계:

IVS는 비디오 스트리밍에 초점이 맞춰져 있어, 스트리밍 외 기능은 별도 구현 필요.

비디오 플레이어 사용자 정의(Customization)는 제한적.



특정 AWS 서비스 의존성:

IVS는 AWS 환경에서만 작동.



녹화 포맷 제한:

HLS 형식으로 녹화된 파일만 제공되므로, MP4 등 다른 형식이 필요하면 별도의 변환 작업 필요.
