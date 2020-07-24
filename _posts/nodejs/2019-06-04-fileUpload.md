---
title: "node.js - File upload 하기"
categories: 
  - node.js
tags:
  - node.js
last_modified_at: 2019-06-04T23:02:00+09:00
author_profile: true
---
**node.js**는 어떻게 파일 업로드를 하는지 찾아봤습니다.

그리고 대부분이 [multer](https://www.npmjs.com/package/multer)라는 모듈을 이용하는 거 같습니다. (fs를 이용한 file upload도 찾았는데 참조 링크를 참조해주세요.)

[Multer](https://www.npmjs.com/package/multer)는 파일 업로드를 위해 사용되는 `multipart/form-data` 를 다루기 위한 **node.js 의 미들웨어**입니다. 효율성을 최대화하기 위해 [busboy](https://github.com/mscdex/busboy)를 기반으로 하고 있습니다.  **by npm-multer**

## 설치

    npm install --save multer
    npm install --save --dev @types/multer # Typescript

## 기본적인 사용 예제

    {% highlight typescript %}
    import express from "express";
    import UploadController from "./UploadController";
    import multer = require("multer");

    export const upload = multer();

    export default express
    .Router()
    .post(
        "/upload",
        [upload.array("files", 10)],
        UploadController.upload
    );
    {% endhighlight %}

위에서는 **array**라고 해서 여러 파일을 받기 위해 사용했지만 `.single()`, `.array()`, `fields()` 이렇게 사용할 수 있습니다. `fields()`는 **다른 key**들을 이용해서 받을 때 사용합니다.


### 응용 1
저는 이미지 파일과 일반 파일을 분류하기 위해서 다음과 같이 사용했습니다.

    {% highlight typescript %}
    import express from "express";
    import UploadController from "./UploadController";
    import multer = require("multer");
    import fileHandler from "../../middlewares/file.handler";
    import moment = require("moment");

    export const fileStorage = multer.diskStorage({
        destination: (req, file, cb) => {
            let date: string = moment().format("YYMMDD");
            if (file.originalname.match(/\.(jpg|jpeg|png|gif)$/)) {
            cb(null, "upload/images/" + date + "/");
            } else if (file.originalname.match(/\.(txt|csv)$/)) {
            cb(null, "upload/files/" + date + "/");
            }
        },
        filename: (req, file, cb) => {
            cb(null, file.originalname);
        }
    });

    export const upload = multer({
    storage: fileStorage, // 위에서 선언한 fileStorage
    limits: {
        files: 10
    }
    });

    // fileHandler는 밑에 설명
    export default express
    .Router()
    .post(
        "/upload",
        [fileHandler, upload.array("files", 10)],
        UploadController.upload
    );

    {% endhighlight %}

개발을 하면서 난관에 부딪힌 부분이 있는데 위의 **destination**의 **req**입니다.

저는 req가 `express.Request` 속성인 줄 알고 Client에서 받은 req를 받을 수 있을 줄 알았는데, 저 속정은 Multer의 `Express.Request` 속성이어서 `req.file`과 `req.files`밖에 못 받았습니다.

![1](/assets/img/posts/nodejs/fileUpload/1.png)

그래서 어쩔 수 없이 확장자 명을 받아서 저장 위치를 따로 분류를 했습니다.

그리고 fileHandler라는 것을 만들어서 상위 폴더가 없으면 폴더를 만들어 주는 로직을 추가해줬습니다.

    {% highlight typescript %}
    import { Request, Response, NextFunction } from "express";
    import fs from "fs";
    import moment = require("moment");

    export default function fileHandler(
    req: Request,
    res: Response,
    next: NextFunction
    ) {
        let date: string = moment().format("YYMMDD");
        let fileDir: string = "upload/files/" + date;
        let imageDir: string = "upload/images/" + date;
        try {
            const stat = fs.lstatSync(fileDir);
        } catch (err) {
            fs.mkdirSync(fileDir, { recursive: true });
            fs.mkdirSync(imageDir, { recursive: true });
        } finally {
            next();
        }
    }
    {% endhighlight %}

### 응용2 (추천)
만약 **file Upload** 동시에 **DB**에 정보를 넣어야 한다면, **file Upload** 따로 **DB create** 따로 한다면 도중 **error** 발생 시 매우 귀찮은 일이 발생합니다.

그래서 **handler**가 아닌 **service** 로직에서 **file Upload** 처리를 하였습니다.

    {% highlight javascript %}
    import multer from "multer";
    import PostController from "./post.controller";
    import express from "express";
    import { responseToJson } from "../../../util/responceToJson";
    
    export const upload = multer({}).array("files");    

    export default express.Router()
    .post("/upload", (req, res, next) => {
                                    upload(req, res, (err) => {
                                        if (err) {
                                            if (err["message"] === "Too many files") {
                                                res.json(responseToJson(403.5, err["message"]));
                                                return;
                                            } else if (err["message"] === "File too large") {
                                                res.json(responseToJson(403.6, err["message"]));
                                                return;
                                            }
                                            res.json(responseToJson(500, err["message"]));
                                            return;
                                        }
                                    next();
                                });
                            }, PostController.uploadPost)
    {% endhighlight %}

**router**에서 요청이 왔을 때, **multer**를 이용한 `multipart/form-data`를 받고 전달을 합니다.

    {% highlight typescript %}
    // PostController.uploadPost

    class PostController {
        uploadPost(req: express.Request, res: express.Response) {
            
            /**
            multer를 이용한 multipart/form-data를 받았기 때문에 req.files 받기 가능

            req.files: Express.Multer.File[]            
            */
            PostService.uploadPost(req.body, req.files)
            .then((r) => {
                res.status(200).send(r);
            })
            .catch((e: Error) => {
                res.status(400).send(e.message);
            });
        }
    }
    {% endhighlight %}    

![2](/assets/img/posts/nodejs/fileUpload/2.png)

`req.files`를 다음과 같이 전달받은 후 `Express.Multer.File[]`에 정의된 타입을 이용하여 사용자에 맞게 구현을 하면 됩니다.

---
#### 참고 및 출처

- <https://victorydntmd.tistory.com/39>

multer
- <https://www.npmjs.com/package/multer>

fs를 이용한 file upload
- <https://infodbbase.tistory.com/59>
