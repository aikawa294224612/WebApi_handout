## Web api 設計與開發_整理
 - xml over http
 - json over http
 - SOAP
 - XML-RPC
 - q: 查詢參數
 - **OAuth2.0**
 - [RFC6838](https://tools.ietf.org/html/rfc6838) 定義自定義media type
 - [RFC7234](https://tools.ietf.org/html/rfc7234)緩存
 - [RFC7230](https://tools.ietf.org/html/rfc7230) HTTP1.1
 - [RFC5789](https://tools.ietf.org/html/rfc5789) patch
 - [RFC6585](https://tools.ietf.org/html/rfc6585) new status code
 - [RFC3339](https://tools.ietf.org/html/rfc3339),[RFC1123](https://tools.ietf.org/html/rfc1123) time standard
 - [RFC6749](https://tools.ietf.org/html/rfc6749) OAuth2.0
 - [ISO3166](https://zh.wikipedia.org/wiki/ISO_3166) 國家名標準
 - [ISO639](https://zh.wikipedia.org/wiki/ISO_639) 語言明標準

**主機明和端點共有部分:api**
ex:

    api.twitter.com/1.1
    api.tumblr.com/v2

###  HATEOAS:
hypermedia as the engine of application state
[https://api.github.com/](https://api.github.com/)

    {
      "current_user_url": "https://api.github.com/user",
      "current_user_authorizations_html_url": "https://github.com/settings/connections/applications{/client_id}",
      "authorizations_url": "https://api.github.com/authorizations",
      "code_search_url": "https://api.github.com/search/code?q={query}{&page,per_page,sort,order}",
      "commit_search_url": "https://api.github.com/search/commits?q={query}{&page,per_page,sort,order}",
      "emails_url": "https://api.github.com/user/emails",
      "emojis_url": "https://api.github.com/emojis",
      "events_url": "https://api.github.com/events",
      "feeds_url": "https://api.github.com/feeds",
      "followers_url": "https://api.github.com/user/followers",
      "following_url": "https://api.github.com/user/following{/target}",
      "gists_url": "https://api.github.com/gists{/gist_id}",
      "hub_url": "https://api.github.com/hub",
      "issue_search_url": "https://api.github.com/search/issues?q={query}{&page,per_page,sort,order}",
      "issues_url": "https://api.github.com/issues",
      "keys_url": "https://api.github.com/user/keys",
      "notifications_url": "https://api.github.com/notifications",
      "organization_repositories_url": "https://api.github.com/orgs/{org}/repos{?type,page,per_page,sort}",
      "organization_url": "https://api.github.com/orgs/{org}",
      "public_gists_url": "https://api.github.com/gists/public",
      "rate_limit_url": "https://api.github.com/rate_limit",
      "repository_url": "https://api.github.com/repos/{owner}/{repo}",
      "repository_search_url": "https://api.github.com/search/repositories?q={query}{&page,per_page,sort,order}",
      "current_user_repositories_url": "https://api.github.com/user/repos{?type,page,per_page,sort}",
      "starred_url": "https://api.github.com/user/starred{/owner}{/repo}",
      "starred_gists_url": "https://api.github.com/gists/starred",
      "team_url": "https://api.github.com/teams",
      "user_url": "https://api.github.com/users/{user}",
      "user_organizations_url": "https://api.github.com/user/orgs",
      "user_repositories_url": "https://api.github.com/users/{user}/repos{?type,page,per_page,sort}",
      "user_search_url": "https://api.github.com/search/users?q={query}{&page,per_page,sort,order}"
    }
rel: 和目前頁面的關係

    {
        "name": "Alice",
        "links": [ {
            "rel": "self",
            "href": "http://localhost:8080/customer/1"
        } ]
    }

**指定數據的方法?**
1. `https://....../?format=xml`
2. `https://....../v1/users.json`
3. 使用Accept request header
`Accept:application/json`

### JSONP
json with padding

### 命名
**json 用駝峰法(userId)**
**性別:**
sex用數值
gender用string
**日期格是用RFC3339 :**

    2015-10-12T11:30:22+09:00
**大整數問題:**
ex: 若id超出long 可再列一個id_str

    {
    "id":377778389990020002222,
    "id_str": "377778389990020002222"
    }

**錯誤:**
1.放在header (私有header)
2.放在body (json)

     {
    "error":{
    "code":2013,
     "message":"bad auth token",
    "info":"http://docs.example.com/api/v1/...."
     }
    }

 **詳細列出給用戶和開發者的訊息:**

     {
     error:{
    "devMessage":".....",
    "userMessage": ".....",
    ....
    }
    }

status code 503返回`Retry-After` header
(可輸出秒數或確切時間)

### 緩存
(fresh/ stale)
RFC7234
**過期模型**
預先決定響應數據綁存期限，當到期後就會再次訪位服務器重新獲取數據
1.response header
(當同時列入時以此為優先)
    Cache-Control: max-age=3600 (sec)

2.response header
不允許超過一年

    Expires: Fri, 01 Jan 2016 00:00:00 GMT

**驗證模型**
輪詢是否微最新數據，但只有在數據更新時才重或數據(如果沒有新的返回304)
1.response header

    Last-Modified: Tue, 01 Jul 2014 00:00:00 GMT
    If-Modified-Since: Tue, 01 Jul 2014 00:00:00 GMT

2.response header -Entity Tag

    ETag: "ff39993jcddjjjj3jj445838999944"
    //可微時間戳或版本號的hash (md5 or sha1)
    If-None-Match: "ff39993jcddjjjj3jj445838999944"

**舉例:**
直接舉一個例子吧，假設我要求 Google 首頁的圖片檔案，收到了這樣的 Response（為了方便閱讀，日期的格式有更改過，實際上的內容不會是這樣）：

    Last-Modified: 2017-01-01 13:00:00  
    Cache-Control: max-age=31536000  

瀏覽器收到之後就會把這張圖片存進快取，並且標明這個檔案的最後更新時間是：`2017-01-01 13:00:00`，過期時間是一年後。

如果在半年後我重新請求這張圖片，瀏覽器就會跟我說：「你不用重新請求喔，這一份檔案的過期時間是一年，現在才過了半年。你要資料是吧？我這邊就有囉！」，於是就不會發送任何 Request，而是直接從瀏覽器那邊獲得資料。

接著我在過了一年之後再請求一次這張圖片，瀏覽器就會說：「嗯嗯，我這邊的快取的確過期了，我幫你去 Server 問一下檔案從`2017-01-01 13:00:00`以後有沒有更新」，會發送出下面這樣的 Request：

    GET /logo.png  
    If-Modified-Since: 2017-01-01 13:00:00  

假設檔案確實更新了，那瀏覽器就會收到一份新的檔案。如果新的檔案一樣有那些 Cache 的 Header，就一樣會快取起來，跟上面的流程都一樣。那假設檔案沒有更新呢？

假設沒有更新的話，Server 就會回一個`Status code: 304 (Not Modified)`，代表你可以繼續沿用快取的這份檔案。

====================================
**不希望緩存時:**
數據變化會對客戶端帶來影響時
數據需要及時時

    Cache-Control: no-cache //不表示不緩存,表示至少需要使用驗證模型來驗證
    Cache-Control: no-store //不緩存
若Expires輸入過期資訊或錯誤日期格式也不會進行緩存

### Cache-Control header

    public ex:天氣api大家訪問都相同
    private  ex:如/users/me獲取自身資訊
    no-cache 
    no-store 
    must-revalidate
    stale-while-revalidate
    max-age

### Media Type (MIME)
Multipurpose Internet Mail Extensions
ex:

    text/plain
    text/html
    application/xml
    application/json
    text/css
    application/javascript
    application/res+xml
    application/zip
    image/jpeg
    image/png
    image/svg+xml
    multipart/form-data
    vedio/mp4
    application/vnd.ms-excel   //Excel
    application/x-www-form-urlencoded

取消Content Sniffing

    X-Content-Type-Options: nosniff

**自定義content type**

    vnd.companyname.awesomefont  //供應商
    prs.   //個人數
    x.    //尚未註冊

### CORS(cross-origin resource sharing)

    Access-Control-Allow-Origin=*
#### Pre-flight Request 事先請求
如:

 - 進行跨域請求前先查詢是否能被接受
 - 使用http方法步是simple methods(head/get/post)
 - 發送時沒帶Accept, Accept Language, Content-Language, Content-Type header
 - Content-Type沒使用下列媒體類型: application/x-www-form-urlencoded, multipart/form-data,  text/plain

事先請求由`OPTION`方法發送

當客戶端使用cookie,或Authentication header發送用戶認證時 (否則會直接拒絕):

    Access-Control-Allow-Credentials:true

### 重構api
**通過版本來管理api (同時提供新舊版api,公告後一段時間再刪除舊版)**
#### 版本
如:

    http://api.tumblr.com/v2/blog/good.tumblr.com/info

有時也會使用日期作為描述api版本號

    https://app.rakuten.co.jp/services/api/InhibaItem/Search/20130805

### 資安
#### Packet Sniffing
#### Session Jacking
#### Https
可使用HSTS (http strict transport security) 限制客戶端只能用https訪問

 - ettercap

#### XSS
將所有`<script>`, `onerror=`跳脫字元
且
確定

	X-Content-Type-Options: nosniff	
    Content-Type: application/json

[https://forum.gamer.com.tw/Co.php?bsn=60292&sn=11267](https://forum.gamer.com.tw/Co.php?bsn=60292&sn=11267)


|<| & lt; |
|--|--|
| > | & gt; |
| " | & quot; |
| ' |& #x27;  |
|/ | & #x2F; |
|& | & amp; |


#### XSRF

 - 設置token驗證
 - 禁止使用get修改數據


#### Json Jacking
json 最外層以object包 {}
[為何 Facebook API 要回傳無窮迴圈？ 談捲土重來的 JSON Hijacking](https://medium.com/@jaydenlin/%E7%82%BA%E4%BD%95-facebook-api-%E8%A6%81%E5%9B%9E%E5%82%B3%E7%84%A1%E7%AA%AE%E8%BF%B4%E5%9C%88-%E8%AB%87%E6%8D%B2%E5%9C%9F%E9%87%8D%E4%BE%86%E7%9A%84-json-hijacking-bc220617ceba)

#### 參數竄改(注意負值)

### 安全header

    X-Content-Type-Options: nosniff	
XSS:

    X-XSS-Protection: 1, mode=block
 Click Jacking
 
    X-Frame-Options:deny
告知不去讀取其他資源，降低XSS

    Content-Security-Policy: default-src 'none'
HSTS: (無法防首次攻擊)

    Strict-Transport-Security: max-age=15768000
Cookie:

    Set-Cookie: Secure; HttpOnly
    
### Rate Limit 單位時間最大訪問量
返回`429 Too Many Requests`
並使用 `Retry-After` header
