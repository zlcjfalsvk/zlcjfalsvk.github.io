---
title: "node.js express + typescript + sequelize(ORM)"
categories: 
  - node.js
tags:
  - node.js
  - express.js
  - typescript
last_modified_at: 2019-05-24T13:00:00+09:00
author_profile: true
---
Java에서는 JPA(Java Persistent API)라고 해서 Java에서 사용할 수 있는 ORM 기술에 대한 API 표준 명세를 의미하는 기술을 이용하며 그를 구현한 ORM 프레임 워크로 대표적으로 [Hibernate](https://hibernate.org/)가 있습니다.

node.js에서는 어떠한 ORM 프레임워크를 이용할까요?<br />
[Sequelize](https://sequelize.org/)가 대표적인 node.js에서 사용하는 ORM 입니다. 

#### Sequelize is a promise-based Node.js ORM for Postgres, MySQL, MariaDB, SQLite and Microsoft SQL Server. It features solid transaction support, relations, eager and lazy loading, read replication and more.
#### Sequelize follows SEMVER. Supports Node v6 and above to use ES6 features.

뭐 보니 DB에 대한 틀 지원, 트랜잭션, 릴레이션, 레이지 로딩 등 지원하고 노드 v6이상부터 지원하며 ES6 기능을 사용한다고 하네요. (SEMVER)가 뭔지는 잘 모르겠습니다.


Sequelize 홈페이지를 가보니 Getting started 부터해서 매뉴얼들이 잘 되어 있더라고요.<br />
저는 Mysql을 이용하기 때문에 Mysql을 이용하여 Connection을 해보겠습니다. 물론 Typescript도 입혀보고요.

우선 이렇게 두 가지를 설치해야 합니다.

    npm install --save sequelize
    npm install --save mysql2

그리고 저는 sequelize를 동시에 @types/sequelize를 설치해야 하는 줄 알았습니다.
하지만 Sequelize는 자체 TypeScript정의를 제공하며 TS>=3.1부터 지원된다 하네요.

그리고 수동으로 다음 세가지는 설치해야 한다고 합니다.

    @types/node
    @types/validator
    @types/bluebird

Connection 자체는 너무 쉽게 설명이 되어 있네요.

    {% highlight typescript %}
    // Default
    import { Sequelize } from "sequelize";
    export const sequelize = new Sequelize(
    config.db_server.database,
    config.db_server.user,
    config.db_server.password,
    {
        host: config.db_server.host,
        dialect: "mysql",
    });

    // 추가적인 옵션들을 넣어봄
    export const sequelize = new Sequelize(
    config.db_server.database,
    config.db_server.user,
    config.db_server.password,
    {
        host: config.db_server.host,
        dialect: "mysql",
        define: {
        timestamps: false  // Sequlize는 기본적으로 Sequelize는 애트리뷰트 
                            // createdAt와 updatedAt모델을 추가하여 데이터베이스 엔트리가 언제 db에 들어 왔는지 그리고 마지막으로 업데이트 된시기를 알 수 있습니다.
        },
        timezone: "+09:00",  // DB Time과 맞춤
        pool: {				 // Connection pool
        max: 30,
        min: 0,
        acquire: 30000,
        idle: 10000
        }
    });
    {% endhighlight %}

**ORM**을 사용하려면 Data를 넣고 받아올 **객체**를 만들어야합니다.

    {% highlight typescript %}
    // NoticeVO.ts
    import { sequelize } from "../configure/connection";
    import Sequelize, { Model } from "sequelize";

    // Sequelize를 사용하기 위해 extends Model<>이게 핵심입니다!
    export class NoticeVO extends Model<NoticeVO> {

    // 저는 Api를 만들면서 Java의 VO처럼 사용하기 위해서 Interface처럼 만들었습니다.
    // VO가 아닌 ORM만 이용하실 경우 밑은 안만들어도 됩니다.
    idx: number;
    title: string;
    content: string;
    type: string;
    leg_dt: Date;
    mod_de: Date;
    }

    // ORM을 이용하기 위한 객체 생성입니다.
    // DB Table 만드는 것과 비슷한 맥락이라 이해하기 쉬우실 겁니다.
    NoticeVO.init(
    {
        idx: {
        type: Sequelize.INTEGER,
        allowNull: false,
        autoIncrement: true,
        primaryKey: true
        },
        title: {
        type: Sequelize.STRING,
        allowNull: false
        },
        content: {
        type: Sequelize.TEXT,
        allowNull: true
        },
        type: {
        type: Sequelize.ENUM("NOTICE", "ETC"),
        allowNull: false
        },
        leg_dt: {
        type: Sequelize.DATE,
        allowNull: false
        },
        mod_dt: {
        type: Sequelize.DATE,
        allowNull: true
        }
    },
    {
        // 제가 만든 sequelize Connection 입니다 Java로 치면 DataSource 같은(?)
        sequelize,
        modelName: "NoticeVO",
        tableName: "c_notice" //사용하는 DB 테이블 명입니다.
    });
    {% endhighlight %}

실제 Sequelize를 이용하여 DB Data를 조회해보겠습니다.
 
    {% highlight typescript %}
    getBoards(page: number = 1): Promise<any> {
        return new Promise((resolve, reject) => {
        NoticeVO.findAll({
            attributes: ["idx","title", "type" ,"leg_dt", "mod_dt"],
            order: [["leg_dt", "DESC"]],
            limit: 10,
            offset: (page - 1) * 10
        }).then(notice => {
            resolve(Utiles.responseToJson(200, notice));
        });
        });
    }
    {% endhighlight %}    

위의 코드를 간단히 설명해 드리겠습니다.

ORM을 사용한 부분은 **NoticeVO.findAll()**입니다. 그리고 **attributes, order** 같은 option을 이용하여 제가 원하는 데이터를 가져올 수 있어요. [옵션](http://docs.sequelizejs.com/manual/models-usage.html#-code-findall--code----search-for-multiple-elements-in-the-database)들은 홈페이지를 참고해주세요.

위의 코드를 실행하면 Sequelize는 다음과 같은 sql을 이용해 Data를 가져옵니다.
#### SELECT idx, title, type, leg_dt, mod_dt FROM c_notice ORDER BY leg_dt DESC LIMIT 10 OFFSET 0

![1](/assets/img/posts/nodejs/sequelize/1.png)

**Data**를 가져온 모습입니다. <br />
**Sequelize**는 Select, Update, Delete, Create 뿐 아니라 여러가지 기능을 제공합니다.
밑의 4가지 기능은 **Sequelize**에서 제공하는 기능입니다.

1. upsert
2. transaction
3. raw query
4. Hooks ( trigger )


### upsert

[**upsert**](https://sequelize.org/master/class/lib/model.js~Model.html#static-method-upsert)라고 들어보셨나요??? 뭔가 합친말 같지 않나요? **insert + update**를 합친거에요.
upsert는 insert하려는 data가 db에 없으면 insert, 있으면 update를 할 수 있도록 해주는 메서드 입니다.
사용방법은 create, findOne 처럼 간단합니다.

	// 메서드 안에는 JSON Oject가 들어가야 합니다. 밑은 제가 그냥 만든 기능이에요.
    // 참고로 upsert는 내부적으로 primary key나 unique key를 찾아서 있으면 update, 없으면 insert를 합니다.
    {% highlight typescript %}
    UserVO.upsert(Utiles.sequelizeToObject(joinUser))
        .then(user => {
            resolve(Utiles.responseToJson(200, token));
        })
       .catch(err => {
            reject(Utiles.responseToJson(500.1, "Data Error"));
        });
    {% endhighlight %}

### transaction

db에서 말하는 transacton이란 작업의 시작부터 해서 commit이나 rollback 까지 이르는 작업의 단위를 transaction이라고 합니다.
그 전에 transaction은 원자성(Atomicity), 일관성(Consistency), 독립성(Isolation), 영구성(Durability)을 보장합니다.
이것을 [ACID](https://ko.wikipedia.org/wiki/ACID)라고 합니다. 각가의 특징은 google에 검색해보세요.

sequelize를 이용하여 [transaction](https://sequelize.org/master/manual/transactions.html)또한 관리할 수 있습니다. 
sequelize를 이용하면 auto commit을 이용할 수도 있고, 개발자가 직접 commit 및 rollback의 작업을 할 수도 있습니다.
auto commit은 manual에 기재되어 있으니 여러개의 쿼리를 실행 하는 동시 트랜젝션을 사용해보겠습니다.

    {% highlight typescript %}
    // 방법 1
	sequelize.transaction(
          {
            isolationLevel: // Mysql transaction 장애 예방 
              Sequelize.Transaction.ISOLATION_LEVELS.READ_COMMITTED
          },
          t1 => { // promise를 이용하거나 cls라는 모듈을 사용할 수 있습니다!
            return Promise.all([
              PointTransferVO.create(
                Utiles.sequelizeToObject(pointTransferVO),
                {
                  transaction: t1
                }
              ),
              PointHistoryVO.create(
                Utiles.sequelizeToObject(pointHistorySender),
                {
                  transaction: t1
                }
              ),
              PointHistoryVO.create(
                Utiles.sequelizeToObject(pointHistoryReceiver),
                { transaction: t1 }
              )
            ]);
          }
        )
        .then(r => {
          resolve(Utiles.responseToJson(200, "success transfer"));
        })
        .catch(err => {
          reject(Utiles.responseToJson(500.1, "failed transfer"));
        });

    // 방법 2
    import { Transaction } from "sequelize/types";

    return new Promise(async (resolve, reject) => {
        const transaction: Transaction = await sequelizeFun.transaction();
        try {
             await CategoryUserVO.destroy({
                where: {
                    user_id: user,
                    category: {
                        [Op.notIn]: category.uncheck_ids,
                    },
                },
                transaction,
            });
            for (const id of category.uncheck_ids) {
                await CategoryUserVO.findOrCreate({
                    where: {
                        user_id: user,
                        category: id,
                    },
                    defaults: { user_id: user, category: id},
                    transaction,
                });
            }
            await transaction.commit();
            resolve("Success");
        } catch (err) {
            await transaction.rollback();
            resolve("fail");
        }
    });    
    {% endhighlight %}

**방법 1**같은 경우는 **transaction t1** 설정 후 Promise.all로 transaction처리할 query들을 실행 시킨 후 만약 catch가 일어나면 rollback처리가 됩니다.

**방법 2**같은 경우는 Transaction을 직접제어하는 방법입니다. 성공했을 시 commit처리를 해주고 실패 시 rollback을 해줘야합니다.

### raw query

ORM을 사용하면 결국 한계가 느껴지는 부분이 있습니다. 저는 join이나 sub query를 작성하면서 느꼈는데요.
sequelize는 ORM 뿐 아니라 직접 [raw query](https://sequelize.org/master/manual/raw-queries.html)를 작성하여 실행할 수도 있습니다.

    {% highlight typescript %}
    sequelize.query(`SELECT SUM(money) as money, name
        FROM (
            SELECT c.TYPE, IF(c.TYPE = "GET", SUM(money), SUM(money) * -1) AS money, u.name
            FROM c_point_history AS c RIGHT OUTER JOIN c_users AS u
            ON c.user_idx = u.user_idx
            WHERE c.user_idx = '${user.user_idx}' GROUP BY c.TYPE
        ) AS b `,
    { type: Sequelize.QueryTypes.SELECT, raw: true }
    ).then(point => {
        resolve(Utiles.responseToJson(200, point));
    });
    {% endhighlight %}

JOIN을 이용하기 위해서 다음과 같은 쿼리를 작성했으며, 밑에 type 지정 및 raw: true를 설정 하였습니다.

### Hooks (TRIGGER)

sql에서 trigger라고 이벤트가 발생되었을 때 해당 sql을 실행시키는 프로시저라고 할 수 있으며

sequelize에서는 [hooks](https://sequelize.org/master/manual/hooks.html)이라고 표현한다고 하는데 아직은 사용해보지 않았으며, 이러한 기능이 있다고 합니다.


---
#### 참고 및 출처

transaction
- https://k39335.tistory.com/28

mysql isolation level
- http://gywn.net/2012/05/mysql-transaction-isolation-level/﻿