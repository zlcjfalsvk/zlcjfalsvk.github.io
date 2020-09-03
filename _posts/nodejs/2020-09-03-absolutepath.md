---
title: "node.js - Typescript Absolute Path"
categories: 
  - node.js
tags:
  - node.js
  - typescript
  - vsCode
last_modified_at: 2020-09-03T21:32:00+09:00
author_profile: true
---
**vsCode**로 **Typescript**로 개발을 하면서 **module import** 를 할 때 **intellisense**가 자동으로 됩니다.

예를들어 다음과 같은 프로젝트가 구성되어 있다 가정 할 때

### 파일 경로

    tsconfig.json # project root path
    src/ # source
      component/
        input/
            input.tsx  # input component
      types/ 
        types.ts # type이 명시되어 있는 폴더

#### types.ts

    {% highlight typescript %}
    export declare module CheolEvent {
        type onChange = React.ChangeEvent<HTMLInputElement>;
        type onClick = React.MouseEvent<HTMLButtonElement, MouseEvent>;
    }
    {% endhighlight %}

### relative path (상대 경로)

**상대 경로**를 이용하면 다음과 같은 코드가 나오게 됩니다.

    {% highlight typescript %}
    import React, { useState } from "react";
    // CheolEvent를 import할 때 intellisense로 인해 상대경로를 가져오게 됩니다.
    import { CheolEvent } from "../../types/types";

    interface Props {
        onClick: (value: string) => void;
    }

    function Input({ onClick }: Props) {
        // source code
    }
    export default Input;
    {% endhighlight %}




### absolute path (절대 경로)

**절대 경로 intellisense**를 사용하기 위해선 `tsconfig.json`에 설정을 추가해줘야 합니다.

    {% highlight json %}
    {
        "compilerOptions": {
            // 생략
            "baseUrl": "./src"
        },
        // 생략 
    }
    {% endhighlight %}

[`baseUrl`](https://www.typescriptlang.org/docs/handbook/module-resolution.html#base-url)을 다음과 같이 추가해주면 import를 할 때 다음과 같이 됩니다.

    {% highlight typescript %}
    import React, { useState } from "react";
    // tsconfig.json을 추가해 절대경로로 모듈을 가져올 수 있습니다.
    import { CheolEvent } from "types/types";

    interface Props {
        onClick: (value: string) => void;
    }

    function Input({ onClick }: Props) {
        // source code
    }
    export default Input;
    {% endhighlight %}



---
#### 참고 및 출처

tsconfig
- <https://www.typescriptlang.org/docs/handbook/tsconfig-json.html>

absolute path
- <https://stackoverflow.com/questions/37606571/absolute-module-path-resolution-in-typescript-files-in-visual-studio-code>
