---
title: "node.js - mysql transaction "
categories: 
  - node.js
tags:
  - mysql
  - node.js
last_modified_at: 2020-08-14T12:13:00+09:00
author_profile: true
---
### transaction

db에서 말하는 transacton이란 작업의 시작부터 해서 commit이나 rollback 까지 이르는 작업의 단위를 transaction이라고 합니다.
그 전에 transaction은 원자성(Atomicity), 일관성(Consistency), 독립성(Isolation), 영구성(Durability)을 보장합니다.
이것을 [ACID](https://ko.wikipedia.org/wiki/ACID)라고 합니다. 

[sequelize orm](https://zlcjfalsvk.github.io/node.js/sequelize/)에서 간단한 **transaction** 사용방법을 소개 했었는데, mysql에서도 **transaction**을 사용하는 방법을 소개해보겠습니다.

[참고](https://zlcjfalsvk.github.io/node.js/mysql/) 과 내용은 비슷합니다.

## 준비

    npm install --save mysql promise-mysql
    npm install --save-D @types/mysql

## 사용법

### mysql.ts - Connection

    {% highlight typescript %}
    export const connection = mysql.createConnection({
    host: config.db_server.host,
    user: config.db_server.user,
    password: config.db_server.password,
    database: config.db_server.database,
    multipleStatements: true  // 다중 쿼리를 실행할 때 설정해줘야 한다.
    });
    {% endhighlight %}

### index.ts - start

    {% highlight typescript %}
    import use from "./use";
    import { connection } from "./mysql";

    (async () => {
        await connection;
        await use.start()
        .then( async (r) => {
            // Connection - end
            (await connection).end().then(() => {
                process.exit(0);
            });
        });
    })();
    {% endhighlight %}

### use.ts - Use

    {% highlight typescript %}
    public async start() {
        try {
            const query = `
            SELECT r.user_id, u.phone_number FROM t_user`

            // transaction 시작
            await (await connection).beginTransaction();

            await (await connection).query(query)
            .then( async (list: selectQuery[]) => {
                for await (const item of list) { 
                    const insert_hot_query = `INSERT INTO relation (object_id, term_taxonomy_id ) VALUES (${item.user_id}, 100);`;
                    const update_hot_query = `UPDATE taxonomy w set w.count = w.count + 1 WHERE term_id = 100;`;     
                    // 다중 쿼리 실행         
                    await (await review_connection).query(`${insert_hot_query} ${update_hot_query}`);
                }
            });
            (await connection).commit();            
        } catch(err) {
            (await connection).rollback();
        } finally {
            await (await review_connection).end();
            return;
        }
    }
    {% endhighlight %}

### 마무리

직접 트랜젝션은 제어 해봤습니다. 만약 `connection pool`을 이용한다면 `connecion.release()`도 해줘야 합니다.


---
#### 참고 및 출처

npm mysql
- <https://www.npmjs.com/package/mysql#transactions>

npm promise-mysql
- <https://www.npmjs.com/package/promise-mysql>