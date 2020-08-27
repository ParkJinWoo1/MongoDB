# MongoDb 

> * MongoDb는 NoSQL 이다

> * 분산처리 가능
> * 데이터 간의 관계를 정의하지 않는 데이터 베이스
> * 고정되지 않은 테이블 스키마(SQL 쿼리문이 없다)
> * 분산형 구조



* interactive
  *  REPL[Read Eval Print Loop(대화형 양방향 shell)]

* javascript interpreter 사용
* js program , library, function 활용 가능, Map Reduce

---



# Collection

**오라클에서의 table**

* document들의 group (rdbms의 table 역할)
* schema를 가지지 않는다 (document들의 field가 각각 다를 수 있다)
  * Collection에 들어가는 데이터들의 형태(형식)을 맞출 필요가 없다
  * schema - 데이터베이스의 구조와 제약조건에 관한 전반적인 명세

# Document

* **{"Key" : "Value"} field : value**

* data recode를 BSON (Binary JSON)으로 저장
* field(key) 중복 불가
* 대소문자 구별



# Primary Key

* _id에 값을 지정해주지 않으면 알아서 id를 만들어준다
* _id는 PK로 본다
* _id : 유일한 식별자



### **function() :** 

```sql
-- mongoDB에서는 Javascript를 활용할 수 있고, function를 활용할 수 있다. 이를 이용한 예제
function find(){
	var tmp = db.score.find().pretty()
	return tmp
}

-- 후에 함수 호출
find()

-- 실행 하면 db.score.find().pretty() 가 실행
```



### find() :

```sql
-- 전체출력
db.score.find()
-- 조건 넣기
db.score.find({name:"이순신"})
-- name에 l 이 포함되어있는 애들 출력
db.score.find({name:/l/})
-- name을 정렬(오름차순)
db.score.find().sort({name:1})
-- name을 정렬(내림차순)
db.score.find().sort({name:-1})
-- 갯수 제한걸기(상단에서 가장 위에 2개 출력)
db.score.find().limit(2)
-- 하나 건너뛰기
db.score.find().skip(1)
-- 배열형태로 이쁘게 출력하기
db.score.find().pretty()
```

### insert() : 

```sql
-- 일반적인 insert
db.score.insert(
    {name:"권율", "kor":100, "eng":100, "math":100}
)

-- 다중 insert
db.score.insertMany([
    {name:"aa", kor:100, eng:100, math:100},
    {name:"bb", kor:100, eng:100, math:100},
    {name:"cc", kor:100, eng:100, math:100}
])

-- 배열형태 insert
db.score.insert(
	{
    	name:"슈퍼맨", buddy:['배트맨', '원더우먼', '아쿠아맨', '조커']
    }
)
```





### update / replace : 

```sql
-- 일반적인 update (하나의 값만) => 가장 맨 위에있는 애
-- name이 홍길동인애를 찾아서, set(바꿔주겠다.) name을 hong으로.
db.score.update(
    {name:"홍길동"},
    {$set:{name:"hong"}
    }
)

-- 다중 update (여러개의 값)
-- name이 홍길동인 애들 전부 찾아서, set(바꿔줄거야) name을 hong으로
db.score.updateMany(
    {name:"홍길동"},
    {$set:{name:"hong"}
    }
)


-- replace
-- score라는 collection에 name이 홍길동인애 컬럼을 다 지우고 class:'bye'만 출력하게바꿔줌
db.score.replaceOne(
    {name:"홍길동"},
    {class:'bye'}
)
```



### $push / $pull / $pop

```sql
-- 추가하기 ($push)
-- name이 '아이언맨'인 애를 update할건데, buddy에 '캡틴아메리카'라고 추가
db.myfriends.update(
    {name:'아이언맨'},
    {$psuh:{buddy:'캡틴아메리카'}
    }
)
-- 값 여러개 배열에 추가
-- name이 '아이언맨'인 애에 추가할건데, buddy에 추가할건데 '캡틴아메리카', '블랙위도우'를 추가
db.myfriends.update(
    {name:'아이언맨'},
    {$push:{buddy:{$each:['캡틴아메리카', '블랙위도우']}}}
)

-- 삭제하기 ($pull)
-- name이 '슈퍼맨'인 애를 update할건데, buddy가 조커인애 pull하기
db.myfriends.update(
    {name:'슈퍼맨'},
	{$pull:{buddy:'조커'}
    }
)

-- 삭제하기 ($pop)
-- 가장 마지막에 있는 애 삭제
db.myfriends.update(
    {name:'슈퍼맨'},
    {$pop:{buddy:1}}
)
```





### delete() : 

```sql
-- 값 하나 삭제하기
-- name이 aa인 애 삭제하기.(조건 걸기)
db.collection.deleteOne({name:"aa"})

-- 전체 삭제
-- 전체 조건
db.collection.deleteMany({})
```



### aggregate : 

|   SQL    |  NOSQL   |
| :------: | :------: |
|  WHERE   |  $match  |
| GROUP BY |  $group  |
|  HAVING  |  $match  |
|  SELECT  | $project |
| ORDER BY |  $sort   |
|  LIMIT   |  $limit  |
|   SUM    |   $sum   |
|  COUNT   |   $sum   |

```sql
-- test가 midterm인 document들 중 국어점수만 찾아서 국어점수의 평균을 test라는 이름으로 출력
-- (_id의 이름은 test의 값들로 한다.)
db.score.aggregate(
    -- stage 1. match(where) : test가 midterm인 애들만.
    {$match:{test:"midterm"}},
    -- stage 2. group(group by) : id는 test의 값으로, average는 kor의 $avg(평균)
    {$group:{_id:'$test', average:{$avg:'$kor'}}}
)

db.score.aggregate(
    -- stage 1. : test가 midterm인 애들만
    {$match:{test:'midterm'}},
    -- stage 2. : select해줄건데 kor과 test만.
    {$project:{kor:1, test:1}},
    -- stage 3. : group해줄건데 _id는 test의 값으로, average는  kor의 값을 avg한 것
    {$group:{_id:'$test', average:{$avg:'$kor'}}}
)
```

### Map-Reduce

aggregate가 처리 못하는 복잡한 집계작업에 사용
javascript function을 사용해서 복잡한 작업처리

**순서** : query => map => reduce => out
**map** : data mapping
**reduce** : 깁계 연산 실행
**query** : 입력될 document
**out** : collection or document 출력



"_id " :  ObjectId("5f45b1d98296ee5716be674c")

_id를 insert를 해주지 않으면(명시하지 않으면) 자동으로 ObjectID값 생성

midterm:{$exists:true}}) midterm존재하면 출력

### 연산자

| 비교연산자 정리 |                                                              |
| :-------------: | :----------------------------------------------------------: |
|      이름       |                             설명                             |
|       $eq       |            값을 비교하여 동일한 값을 찾는 연산자             |
|       $gt       | 값을 비교하여 지정된 값보다 큰 값을 찾는 연산자$gt : 80 => 80초과 |
|      $gte       | 값을 비교하여 지정된 값도나 크거나 같은 값을 찾는 연산자 $gte : 80 => 80이상 |
|       $in       |       배열 안의 값을 비교하여 동일한 값을 찾는 연산자        |
|       $lt       | 값을 비교하여 지정된 값보다 작은 값을 찾는 연산자 $lt : 80 => 80미만 |
|      $lte       | 값을 비교하여 지정된 값보가 작거나 같은 값을 찾는 연산자 $lte : 80 => 80이하 |
|       $ne       |         값을 비교하여 동일하지 않은 값을 찾는 연산자         |
|      $nin       |    배열 안의 값을 비교하여 동일하지 않는 값을 찾는 연산자    |

| 논리연산자 정리 |                                                              |
| :-------------: | :----------------------------------------------------------: |
|      이름       |                             설명                             |
|      $and       |      배열 안 두 개 이상의 조건이 모두 참인 경우를 반환       |
|      $not       | 해당 조건이 맞지 않는 경우와 해당 필드가 없는 경우를 모두 반환 |
|      $nor       |      배열 안 두 개 이상의 조건이 모두 아닌 경우를 반환       |
|       $or       |      배열 안 두 개 이상의 조건 중 하나라도 참이면 반환       |







# 기본 명령어

* use test : test의 공간 확인
* show dbs : 현재 db목록 확인
* db : 현재 db 확인
* use dname : 해당 db로 변경
* show collections : 현재 db의 collection목록 확인





연산자 사용법 {<field>:{<opr>:<value>}}





