---
title: "vsCode - Workspace extensions 관리"
categories: 
  - else
tags:
  - vscode
  - vscode extensions
last_modified_at: 2020-07-13T22:18:00+09:00
author_profile: true
---
[vsCode](https://code.visualstudio.com/)를 이용하여 개발을 할 때 **project**에 필요한 **extensions**를 일일이 설치해야합니다. 이러한 귀찮은 작업을 없애기 위하여 vsCode는 다음과 같은 방법을 제공합니다.


1. 설치할 **extension**의 **identifier**를 알아냅니다.

![1](/assets/img/posts/else/extensions/1.png)

2. **workspace**에 **.vscode**폴더를 만들고 **extensions.json**파일을 만듭니다.

![2](/assets/img/posts/else/extensions/2.png)

    {% highlight json %}
    // extensions.json

    {
        "recommendations": [
        "vscjava.vscode-java-pack"
        ]
    }    
    {% endhighlight %}

3. 파일작성 후 다시 **vscode**를 열어 **extension menu**를 열면 우측하단에 필요한 extensoins이 나옵니다.

![3](/assets/img/posts/else/extensions/3.png)

![4](/assets/img/posts/else/extensions/4.png)


이런식으로 진행을 하면 여러 PC에서 개발을 하는 경우, extension List만 있으면	
[Settings Sync](https://marketplace.visualstudio.com/items?itemName=Shan.code-settings-sync)
를 이용 안해도 간편하게 extension을 설치 할 수 있다는 점인거 같습니다.

---
#### 참고 및 출처

Workspace recommended extensions

- https://code.visualstudio.com/docs/editor/extension-gallery#_workspace-recommended-extensions
