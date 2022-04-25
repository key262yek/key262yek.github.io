---
title: "Develop Rust skill server of kakao chatbot on AWS lambda"
date: 2022-01-30-T14:15:30-14:30
categories:
  - Programmings
tags:
  - Rust
  - Rest_API
  - AWS
toc: true
toc_label: "Table of Contents"
toc_icon: "fas fa-clipboard-list"
toc_sticky: true
---

카카오톡 챗봇 서비스에서는 '스킬'이란 기능이 있습니다. 
챗봇에서 넘어온 http 요청을 통해 데이터를 검증하거나, 챗봇에서 사용할 데이터를 가공해 http 응답을 보내는 일종의 rest api입니다. 
우연히 지인의 회사에서 사용할 챗봇의 스킬 서버를 개발할 일이 생겼습니다. 
이 포스트에서는 해당 스킬 서버를 어디에, 어떻게 만들었는지, 그 과정에서 어떤 문제가 있었는지를 기록해두려고 합니다. 

### Purpose
스킬 서버에서는 다양한 일을 수행할 수 있지만, 제가 만드려는 스킬의 목적은 딱 한 가지였습니다. 
챗봇을 이용하는 사람이 회사의 직원인지 아닌지 여부를 검증하는 기능이 필요했습니다. 
처음에는 챗봇 자체에서 검증기능이 있지 않을까 싶었지만 기본적으로 챗봇은 불특정 다수를 향해 열려있는 것을 전제로 하고 있는지, 특정 집단만을 위한 인증 기능을 제공하고 있지는 않았습니다.
그래도 롯데 사내 챗봇이라던가, 다른 챗봇들에서 분명히 사원 인증을 하고 있는 것으로 보아 분명 길은 있을거라 보았는데요. 
챗봇의 인증 블록에서 스킬을 사용하면 사용자 정보를 확인할 수 있어, 사원 데이터와 대조해 검증 기능을 구현할 수 있겠다 싶었습니다. 
사원 데이터는 사내 전산서버를 이용할까도 했는데 사원도 아닌데 전산팀과 소통할 수 없으니 자체적으로 사원 데이터를 구축하기로 했습니다. 

정리하자면 크게 세 단계의 기능을 구현했습니다. 
 1. 사원 데이터 구축 및 업데이트 자동화 
 2. 챗봇의 사용자 정보 확인 
 3. 사용자 정보와 사원 데이터 대조 
 
### Local? Serverless? : AWS Lambda + DynamoDB
먼저 위의 기능을 구현할 서버를 결정해야 합니다. 
로컬 자원이 있다면 모르겠지만 챗봇을 위해 컴퓨터를 사는 것도 이상한 일이니 요즘 저렴하게 잘 나왔다는 AWS Lambda를 써보기로 했습니다. 
AWS Lambda는 1백만 요청당 수 달러 정도밖에 안 나오기 때문에, 해봐야 월 수 천건 정도 사용될 사내 챗봇에서는 거의 비용 없이 사용할 수 있습니다. 
Python을 web에서 곧바로 코드 수정할 수 있고 테스트도 할 수 있어서 개발하기도 쉬웠습니다. 

이왕 Lambda를 사용하기로 한 김에 사원 데이터를 저장할 DB 역시 AWS에서 찾았습니다. 
Lambda보다 DB에 대해서는 더 모르기 때문에, 그저 무료로 사용할 수 있는 DynamoDB를 이용하기로 했습니다. 
적합한 DB였는지 판단은 잘 안되지만 일단은 크게 불만족스럽진 않았습니다. 

### 사원 데이터 업데이트 자동화
사원이 몇 백명은 되는 회사라 사원 데이터를 업데이트하는 것은 쉬운 일이 아닐겁니다. 
지인이 이를 하나하나 DB를 열어 관리하는 것도 비효율적이고, 처음 DB를 구현할 때도 하나하나 업로드할 수 없으니 데이터를 업데이트할 Lambda 함수를 제일 먼저 구현했습니다. 
크게 데이터를 csv 파일로 정리해 S3에 올리면 lambda로 이를 읽어 DynamoDB에 업데이트 하는 함수입니다. 

먼저 데이터를 올릴 데이터 Table을 DynamoDB에 만들었습니다. 
이후에 핸드폰번호를 기준으로 사원 정보와 대조하려 하니 핸드폰 번호를 담은 문자열을 primary key로 설정했습니다. 
그리고 S3에 버켓을 만들어 csv 파일을 업로드 합니다. 

Lambda에 Python 함수를 하나 만들고, 함수에 S3 읽기 권한과 DynamoDB 쓰기 권한을 부여합니다. 
``` json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "dynamodb:BatchWriteItem",
                "dynamodb:PutItem",
                "dynamodb:UpdateItem"
            ],
            "Resource": [
                "arn:aws:s3:::{BucketName}",
                "arn:aws:s3:::{BucketName}/*",
                "arn:aws:dynamodb:ap-northeast-2:{AccountID}:table/{TableName}"
            ]
        }
    ]
}
```

Python 코드는 아래와 같습니다. 
``` python
import json
import boto3 # AWS 접근 패키지
import csv
    
def lambda_handler(event, context):
    # Open S3 client
    s3 = boto3.client('s3')
    
    # Open DynamoDB client
    db = boto3.client('dynamodb')
    
    # S3에서 {Bucket Name} 버켓에서 {File name}.csv 파일을 읽어옵니다. 
    response = s3.get_object(Bucket='{Bucket Name}', Key='{File name}.csv')
    
    # 불러온 데이터에서 Body(streamBody) 항목을 불러와 파일핸들로 읽듯이 읽어와 라인별로 잘라 저장한다.
    rec_data = response.get('Body').read().splitlines()
    
    # 데이터가 바이트 형태로 되어있기 때문에 문자열 형태로 변환해준다.
    # 인덱스값(idx)이 0이면 첫번째줄로 타이틀이기때문에 넘어간다.
    rec_list = [x.decode(encoding = 'utf8') for x in rec_data[1:]]
    
    # 콤마(,)를 기준으로 csv 파싱
    # , 를 기준으로 항목을 구분하되 " 로 감싸여 있는 것은 , 가 있더라도 무시한다.
    csv_reader = csv.reader(rec_list, delimiter=',', quotechar='"')
    
    # 파싱된 데이터를 줄(row)단위로 인덱스값(idx)과 함께 루프를 돌린다.
    for idx, row in enumerate(csv_reader):
        name = row[1]   # 이름
        pos = row[2]    # 직급
        dept = row[3]   # 부서
        phone = row[4]  # 휴대전화
        email = row[5]  # 이메일
        
        if phone == '':
            continue
        
        # dynamodb에 쓴다.
        # 만약 DB에 존재하는 primary key라면 Item을 업데이트하는 역할을 합니다. 
        response = db.put_item(
            TableName='{Table Name}',
            Item= {
                'phone':{'S':phone},
                'name':{'S':name},
                'dept':{'S':dept},
                'email':{'S':email},
                'position':{'S':pos},
            }
        )
    # 완료
    print('succeeded')
```

애초에 이 코드만 실행하면 데이터를 업데이트할 수 있는데 한 가지 문제가 남습니다. 
가장 편한 실행 방법은 웹에서 테스트를 실행하는 것인데, 실행 제한시간이 짧아서 csv를 한번에 업로드하지 못합니다. 
Timeout은 기본 3초로 설정되어 있어서, 이를 늘려줄 필요가 있습니다. 
Lambda 함수의 Configuration 항목에서 Timeout을 10분 정도로 넉넉하게 설정하면 됩니다. 

### 챗봇 사용자 정보 확인
이제 새로운 lambda 함수를 만들어서 챗봇의 인증 요청을 수행하는 기능을 부여해봅시다. 
먼저 DB와 대조할 사용자 정보를 얻을 필요가 있습니다. 
챗봇의 인증블록에서 스킬로 전달되는 요청에는 사용자 정보를 받을 수 있는 otp key가 담겨져있습니다. 
``` json
{
    ...
    "origin": "https://talk-plugin-capi.kakao.com/otp/62247d59d36c554596ea60eb/profile",
    "value": "{\"otp\":\"https://talk-plugin-capi.kakao.com/otp/62247d59d36c554596ea60eb/profile\",\"app_user_id\":2113105765}"
    ...
}
```
이 otp key에 챗봇 관리자의 rest api key를 붙여서 kakao에 요청하면 사용자 이름, 번호, 프로필 사진 주소 등을 다시 응답해줍니다. 
otp key를 파싱해 사용자 정보를 반환하는 함수는 아래와 같이 쓸 수 있습니다. 
``` python
def parse_userinfo_api(event):
    # event : lambda에 전달된 http request
    body = json.loads(event['body'])
    otp = body['value']['origin']
    
    return otp

def get_user_info(otp):
    url = otp + "?rest_api_key={rest_api_key}"
    http = urllib3.PoolManager()
    response = http.request('GET', url)
    data = json.loads(response.data.decode('utf-8'))
    name = data['nickname']
    phone = '0'+ data['phone_number'][4:]. # +82-10-2222-3333 꼴에서 010-2222-3333으로 수정
    
    return (name, phone)
```

### DB의 사원 데이터와 사용자 정보 대조
사용자의 핸드폰 번호를 알게 되었으니 DB와 대조해 결과를 응답해줄 일만 남았습니다. 
DB의 읽기 권한이 필요하니 lambda 함수의 권한을 아래와 같이 수정해줍니다. 
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "dynamodb:BatchGetItem",
                "dynamodb:GetItem",
                "dynamodb:Scan",
                "dynamodb:Query",
            ],
            "Resource": [
                "arn:aws:dynamodb:ap-northeast-2:{Account ID}:table/{Table Name}"
            ]
        }
    ]
}
```

그리고 DB에 사용자 핸드폰 번호를 primary key로하는 사원이 있는지 확인하는 함수를 아래와 같이 적어줍니다. 
```python
def return_data_api(phone):
    dynamodb = boto3.client('dynamodb')
    exist = dynamodb.get_item(TableName = '{Table Name}', Key = {'phone':{'S' : phone}})
    if 'Item' in exist:
        return {
            'statusCode': 200,
            'body' : json.dumps({
                "version" : "2.0",
                "status" : "SUCCESS"
            }),
        }
    else:
        return {
            'statusCode': 200,
            'body' : json.dumps({
                "version" : "2.0",
                "status" : "FAIL"
            }),
        }
```
스킬에서 데이터를 전달하는 경우도 있지만 여기서는 인증의 성공, 실패 결과만 전달해주면 되기 때문에 위와 같은 응답으로도 인증이 잘 작동합니다. 
이런 함수들을 종합해 전체 코드는 맨 앞과 같이 쓰입니다.


### Cold start 문제 - Rust로 변환
Python으로도 함수가 잘 기능하긴 하지만 한가지 꽤 큰 문제가 있었습니다. 
Lambda는 일정 시간 이상 요청이 없으면 함수를 재워두다가, 다시 요청이 오면 재실행하는 방식으로 되어있습니다. 
그래서 오랜만에 요청하는 경우 응답시간이 0.5초에서 1.5초로 늘어났고, 챗봇은 1초 가량을 넘기면 응답지연으로 보기 때문에 인증실패 메시지가 뜨게됩니다. 
문제는 10분에서 20분만 요청이 없어도 cold start하기 때문에 인증실패가 제법 자주 생긴단 점이었습니다. 

그래서 혹시 Python에서 Rust로 바꾸면 compile 과정이 없어도 되니 더 빨리 구동할까 싶었습니다. 
실제 실행시간도 더 빠를테니 cold start 하더라도 timeout 전에 응답할 수 있지 않을까 싶었구요. 
결과적으로는 warm start의 경우엔 실행시간이 0.5초에서 0.2초로 줄었고, cold start는 1.5초에서 0.9초로 줄었습니다. 
이러면 해결될거라 생각했는데, 안타깝게도 카카오 챗봇은 이마저도 timeout 처리했습니다. 
결국 챗봇에서 인증실패에 대한 재응답 요청횟수를 3회로 늘려 해결하는 방식을 썼습니다. 

### Rust 실행파일 업로드
Rust는 AWS lambda에서 곧바로 코드 수정을 할 수 없습니다. 
빌드한 실행파일을 정해진 규격에 맞춰 lambda에 업로드하면 되는데 이 부분이 생각보다 까다로웠습니다. 
첫번째로 lambda에서 사용하는 os에 맞춰 `x86_64-unknown-linux-gnu`를 target으로 cross-compile 해야합니다. 
그를 위한 세팅은 아래의 명령어로 가능합니다. 
```
rustup target add x86_64-unknown-linux-gnu
cargo install cross
``` 
이 상태에서 cargo로 빌드하려면 아래와 같이 하면됩니다. 
```
cargo build --release --target x86_64-unknown-linux-gnu 
``` 
대신 이 경우엔 실행파일을 bootstrap.zip라는 이름으로 압축해 업로드해야합니다. 
이 과정을 한번에 하려면 cargo-lambda를 이용하면 됩니다. 
```
cargo install cargo-lambda
cargo lambda build --release --target x86_64-unknown-linux-gnu --output-format zip
```
cargo-lambda는 이뿐 아니라 lambda 함수를 테스트할 수 있는 환경을 제공해주기도 합니다. 
lambda 함수는 기본적으로 서버에서 구동되며 요청을 받아 응답하는 함수이기 때문에, 로컬 서버 환경을 구현해야지만 테스트할 수 있습니다. 
`cargo lambda start` 명령어를 통해 로컬 서버를 세팅하고 `cargo lambda invoke` 명령어를 통해 함수에 요청을 보내게 됩니다. 

### 함수 변환
Python으로 적힌 함수를 Rust로 옮기는 과정은 쉽지 않았습니다. 
전부를 다 적는 것은 불필요하니 중요한 문제들만 나열해봅니다. 

#### Dependency problem
Python으로 쓸 때보다 Rust로 lambda 함수로 작성하기 위해서는 꽤나 많은 crate를 이용해야했습니다. 
비동기 함수로 쓰여져야하기 때문에 tokio crate, 
AWS-SDK인 boto3 패키지는 aws-sdk-dynamodb, lambda-runtime, lambda-http crate, 
JSON 파서로는 serde, serde_json crate,
http 요청을 위해서는 reqwest crate를 이용해야합니다. 
여기서 reqwest crate는 제대로 의존성을 관리하지 않으면 openssl crate의 버젼 문제 때문에 compile 에러가 생깁니다. 
아예 reqwest 버젼을 낮추는 방법도 고려해봤지만, 전혀 통하지 않았고 feature를 아래와 같이 바꾸는 방법만이 유일한 해법이었습니다. 
```toml 
reqwest = { version = "0.11.10", features = [ "json", "rustls-tls" ], default-features = false }
```

#### Quotation in JSON
두번째 문제로 따옴표 문제가 있었습니다. 
Python에서는 따옴표를 작은 따옴표, 큰 따옴표를 사용하기 때문에 JSON 형식을 표현하는 것에 문제가 없었습니다만, 
Rust에서 작은 따옴표는 오직 단일 문자를 위해서만 쓰이고, 문자열에는 큰 따옴표만 쓰이기 때문에 따옴표가 포함되는 JSON 형식을 표현하기에 적합하지 않습니다. 
때문에 문자열 내부에 확장문자가 포함되고, 이를 제대로 다루지 않게되면 적합한 응답으로 처리되지 않게 됩니다. 
이런 경우 아예 확장문자를 제거하거나 snaiquote의 unescape 함수가 쓰여야합니다. 
```rust 
use snailquote::unescape;

fn parse_otp(event : Value) -> Result<String, Box<StringError>>{
    let mut otp = event["value"]["origin"].to_string();
    otp.retain(|c| c != '\"');
    Ok(otp)
}

async fn get_user_info(mut otp : String) -> Result<(String, String), Box<StringError>>{
    otp.push_str("?rest_api_key={Rest API key}");
    let res = reqwest::Client::new().get(otp).send().await.map_err(|e| StringError::new(format!("OTP request failed with error : {:?}", e)))?
                                    .text().await.map_err(|e| StringError::new(format!("Parse response to text failed with error : {:?}", e)))?;
    let data : Value = from_str(&res).unwrap();
    let name = unescape(&data["nickname"].to_string()).unwrap();
    let phone = unescape(&data["phone_number"].to_string().replace("+82 ", "0")).unwrap();

    return Ok((name, phone));
}
```

#### Result
crate 버젼과 최종 결과물은 아래와 같습니다. 
```toml 
[dependencies]
tokio = { version = "1.17", features = ["full"] }
serde = "1.0.100"
serde_json="1.0.79"
log = "0.4.16"
lambda_runtime = "0.5.1"
lambda_http = "0.5.1"
http = "0.2"
aws-config = "0.10.1"
aws-sdk-dynamodb = "0.10.1"
reqwest = { version = "0.11.10", features = [ "json", "rustls-tls" ], default-features = false }
snailquote = "0.3.1"
```

```rust
use aws_config::meta::region::RegionProviderChain;
use aws_sdk_dynamodb::{Client, model::AttributeValue};
use lambda_runtime::service_fn;
use lambda_http::{Response, Request, Body, RequestExt};
use http::status::StatusCode;
use std::convert::Into;
use serde_json::{Value, from_str};
use reqwest;
use snailquote::unescape;

#[derive(Debug)]
struct StringError{
    body : String,
}

impl StringError{
    pub fn new(s : String) -> Box<Self>{
        Box::new(Self{
            body : s,
        })
    }
}

impl std::fmt::Display for StringError{
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        write!(f, "{}", self.body)
    }
}

impl std::error::Error for StringError {}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error + Send + Sync + 'static>> {
    let func = service_fn(func);
    lambda_http::run(func).await.map_err(|e| StringError::new(format!("Lambda run failed with error : {:?}", e)))?;
    Ok(())
}

async fn func(req: Request) -> Result<Response<&'static str>, Box<dyn std::error::Error + Send + Sync + 'static>> {
    let event : Value = match req.body(){
        Body::Text(s) => from_str(s).unwrap(),
        _ => {return Err(StringError::new("Request body is mal-formed".to_string()));}
    };
    let _context = req.lambda_context();

    let region_provider = RegionProviderChain::default_provider().or_else("ap-northeast-2");
    let config = aws_config::from_env().region(region_provider).load().await;
    let client = Client::new(&config);

    let otp = parse_otp(event)?;
    let (name, phone) = get_user_info(otp).await?;

    let res = check_exist(&client, phone).await?;
    let resp = response_builder(res).unwrap();

    Ok(resp)
}

async fn check_exist<T>(client : &Client, phone : T) -> Result<bool, Box<StringError>>
    where T : Into<String> + Clone{
    let resp = client.get_item().table_name("{Table Name}")
                     .key("phone", AttributeValue::S(phone.into()))
                     .send().await.map_err(|e| StringError::new(format!("DB function failed with error : {:?}", e)))?;
    let item = resp.item;
    match item{
        Some(_) => Ok(true),
        None => Ok(false),
    }
}

fn parse_otp(event : Value) -> Result<String, Box<StringError>>{
    let mut otp = event["value"]["origin"].to_string();
    // println!("Raw otp : {}", otp);
    otp.retain(|c| c != '\"');
    Ok(otp)
}

async fn get_user_info(mut otp : String) -> Result<(String, String), Box<StringError>>{
    otp.push_str("?rest_api_key={Rest API key}");
    let res = reqwest::Client::new().get(otp).send().await.map_err(|e| StringError::new(format!("OTP request failed with error : {:?}", e)))?
                                    .text().await.map_err(|e| StringError::new(format!("Parse response to text failed with error : {:?}", e)))?;
    let data : Value = from_str(&res).unwrap();
    let name = unescape(&data["nickname"].to_string()).unwrap();
    let phone = unescape(&data["phone_number"].to_string().replace("+82 ", "0")).unwrap();

    return Ok((name, phone));
}

fn response_builder(res : bool) -> http::Result<Response<&'static str>>{
    let builder = Response::builder().status(StatusCode::OK);
    let body = match res {
        true => {
            r#"{"version" : "2.0","status" : "SUCCESS"}"#
        },
        false => {
            r#"{"version" : "2.0","status" : "FAIL"}"#
        },
    };
    builder.body(body)
}

```



