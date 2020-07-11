---
title: "Angular - File Upload"
categories: 
  - Angular
tags:
  - Angular
last_modified_at: 2018-08-08T17:04:00+09:00
author_profile: true
---
웹 애플리케이션 파일 업로드는 크게 두가지의 방식이 있다.

1. multipart/form-data

    **FormData**객체를 사용하여 \<**input type="file"**\> 요소로 부터 취득한 file 정보를 **append**하여 서버로 전송하는 방식이다.

2. applecation/x-www-urlencoded

    클라이언트는 바이너리 파일을 Base64 인코딩하여 문자열화한 후, 서버로 전송하고 서버는 **Base64** 인코딩된 문자열을 디코딩하여 저장하는 방식이다. 인코딩으로 인한 성능 이슈가 발생할 수 있다.

**applecation/x-www-urlencoded** 방식은 인코딩으로 인한 성능 이슈가 발생할 수 있다. <br />
**multipart/form-data** 방식을 사용하여 파일을 전송하는 예제를 작성하여 보자.

### View
    {% highlight html %}
    <!-- 버튼으로 파일추가 구현하려면 input의 style에 display: none;하고 템플릿 참조변수를 이용해 대신 클릭하게 해준다. -->
    <input type="file" name="image" (change)="onFileChange($event)" multiple="multiple" accept="image/gif, image/jpeg, image/png" #fileInput>

    <!-- 템플릿 참조변수 -->
    <button type="button" (click)="fileInput.click();">이미지 추가</button>    
    {% endhighlight %}

### Component
    {% highlight typescript %}
	onFileChange(fileInput) {
		let urlName;
		const files: Array = >fileInput.target.files; // multiple로 인해 여러 파일을 선택할 경우에는 Array로 처리

		Array.from(files).map(file => {
			const formData: any = new FormData(); // FormData 이용
			formData.append('file', file);			
                        ......
                       // 파일을 처리하는 로직 장성
	}    
    {% endhighlight %}


---
#### 참고 및 출처
