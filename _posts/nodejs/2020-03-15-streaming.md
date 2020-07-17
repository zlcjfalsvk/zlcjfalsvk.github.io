---
title: "node.js - AWS 스트리밍 다운로드 (동영상 스트리밍)"
categories: 
  - node.js
tags:
  - node.js
  - Streaming
  - AWS
last_modified_at: 2020-03-15T19:04:00+09:00
author_profile: true
---
AWS에 올린 File을 스트리밍 형식을 통하여 다운로드하는 방식을 이용해보겠습니다.

**스트림**이라는 뜻을 찾아봤습니다. [이동](http://www.ktword.co.kr/word/abbr_view.php?m_temp1=1311)
해당 링크에서는 다양한 분야에서 **스트림**이라는 용어를 이용하지만, 

제가 이용할 스트림은 **프로그래밍 관점에서의 스트림**으로써 물리 디스크 상의 파일, 장치들을 통일된 방식으로 다루기 위한 가상적인 개념으로 알고 있으면 될 거 같습니다.

프로그래밍 언어마다 Stream을 지원하며 **node.js** 또한 **stream**을 제공합니다.
예를 들어 file을 읽기/쓰기를 할 때는 **createReadStream & createWriteStream**을 이용할 수 있습니다. 물론 **파이프라인**도 제공합니다.

직접적인 파일을 Read, Write를 이용하면 **fs(File System)**을 이용하면 되지만, **AWS**를 이용한다면 **aws-sdk**를 이용하면 됩니다.

### 예제

    {% highlight typescript %}  
    // stored_file_path: AWS s3에 있는 파일 위치 Key
    public getImages(stored_file_path: string, res: express.Response) {
    
      const getParam = {
        Bucket: config.AWS.S3.BUCKET, // 버킷 위치
        Key: `${stored_file_path}`,
      };
      
      // headObject는 파일이 있나 없나 먼저 체크 용도로 이용
      s3.headObject(getParam, (err, data) => {
        if (err) { // 없을 경우 에러처리
          res.status(200).json(responseToJson(500, err.message));
        } else {
          // Http 통신을 한다면 Response 해더를 설정해 줘야 합니다.
          res.contentType("image/*"); // Contents-Type 설정
          // 파일명 설정
          res.setHeader(`Content-Disposition`, `attachment; filename=${stored_file_path.split("/")[3]}`);
          s3.getObject(getParam).createReadStream()
          .pipe(res)
          .on("error", (err) => { return; })
          .pipe(res)
          .on("close", (err) => { return; });
        }
      });
    }    
    {% endhighlight %}

위의 코드 같은 경우는 스트림을 이용한 이미지 파일다운로드 입니다. 

이런식으로 파이프 라인 스트림을 이용하여 데이터를 전송할 수 있으며, 동영상 스트리밍 같은 경우에도 이러한 경우를 이용하여 전송하면 됩니다. 

하지만 스트리밍을 하기 위해서는 규약이 있는데요.

1. response 에 Content-Range가 있어야 한다.
2. 처음에는 200 response를 받고 그 이후 206을 지속적으로 줘야 합니다.

(HTTP **206 Partial Content**는 [Range](https://developer.mozilla.org/ko/docs/Web/HTTP/Headers/Range) 헤더에 기술된 데이터 범위에 대한 요청이 성공적으로 응답되어 바디에 해당되는 데이터를 담고 있다는 것을 알려줍니다. [MDN Web Docs](https://developer.mozilla.org/ko/docs/Web/HTTP/Status/206))

  - 자료조사 하기위해 유튜x, 네이x, 넷x릭스, 카카x를 확인해 보니  카카x를 빼고는 계속 200을 전해 주던데 그 이유는 뭘까요? => 개인적인 생각으로는 스트림으로 계속 전송이 아닌 분산처리로 인해 request 요청으로 분할 방식으로 주는거 같은데 맞는지 모르겠습니다. 참고로 유튜x는 204로 줌

아무튼 저는 **aws-sdk**를 이용한 S3에 있는 영상을 스트리밍을 이용하여 가져오는 코드를 작성해보겠습니다.

    {% highlight typescript %}
    public getObjectStream(stored_file_path: string, res: express.Response, range: { start: number, end: number }, first: boolean) {
      const getParam = {
          Bucket: config.AWS.S3.BUCKET,
          Key: stored_file_path,
        };

      s3.headObject(getParam, (err, data) => {
          if (err) {
            res.status(200).json(responseToJson(500, err.message));
          } else {
            res.contentType("video/mp4"); // contents-type 설정 
            if (first) {
              // aws-sdk의 파일을 가져올 때 이용
              range.end = range.end !== 0 ? range.end : (data.ContentLength as number - 1);
              const chunksize = (range.end - range.start) + 1;
              const bytes: string = `bytes ${range.start}-${range.end}`;
              res.status(206);
              res.setHeader("Accept-Ranges", "bytes");
              res.setHeader("Content-Range", bytes + "/" + data.ContentLength?.toString());
              res.setHeader("Content-Length", chunksize);
              // 중요! stream을 이용하여 가져올 때 range를 설정 안해주면 스트리밍이 안된다.
              getParam["Range"] = `bytes=${range.start}-${range.end}`;
            } else {
              res.status(200);
              res.setHeader("Content-Length", data.ContentLength as number);
            }
            s3.getObject(getParam).createReadStream()
            .on("error", (err) => { return; })
            .pipe(res)
            .on("close", (err) => {  return; });
          }
        });
    }    
    {% endhighlight %}

**res.setHeader** 부분이 **response**를 할 때 설정해줘야 하는 값 들이며, **getParam\["Range"\]**는 **aws-sdk**에서 **Stream**을 이용하며 파일을 가져올 때 설정해 줘야 합니다. (이것때문에 삽질함ㅠㅠ)

그 이후 **pipe()**를 이용하여 파이프 라인으로 전송을 해주었으며, **on** 같은 경우에는 에러 처리를 위해 설정 해줬습니다.


---
#### 참고 및 출처

스트림이란?
- http://www.ktword.co.kr/word/abbr_view.php?m_temp1=1311

파이프라인
- https://ko.wikipedia.org/wiki/%ED%8C%8C%EC%9D%B4%ED%94%84%EB%9D%BC%EC%9D%B8_(%EC%BB%B4%ED%93%A8%ED%8C%85)

aws node.js stream
- https://docs.aws.amazon.com/ko_kr/sdk-for-javascript/v2/developer-guide/requests-using-stream-objects.html

node.js File System Stream
- https://nodejs.org/api/fs.html#fs_fs_createreadstream_path_options

Streaming Example
- http://commondatastorage.googleapis.com/gtv-videos-bucket/sample/ElephantsDream.mp4

Streaming Header
- https://medium.com/better-programming/video-stream-with-node-js-and-html5-320b3191a6b6
- https://www.3rabbitz.com/blog_ko/d9b3b3b10d2fee4a
- https://stackoverflow.com/questions/53226595/streaming-audio-in-node-js-with-content-range

206 respnose
- https://developer.mozilla.org/ko/docs/Web/HTTP/Status/206﻿
