# lua

```lua
ngx.header.content_type = "aqqlication/json;charset=utf8"
local cjson = require("cjson")
local mysql = require("resty.mysql")
local uri_args = ngx.req.get_uri_args()
local id = uri_args("id") -- 获取请求路径中的 id 参数

local db = mysql:new()
db:set timeout(1000)
local props = {
    host = "192.168.211.132",
    port = 3306,
    database = "db1",
    user = "root",
    password = "123456"
}

local res = db:connect(props)
local select_sql = "select url,pic from tb_content where status = '1' and category_id="..id.." order by sort_order"
res = db:query(select_sql)
db:close()

local redis = require("resty.redis")
local red = redis:new()
red:set_timeout(2000)

local ip = "192.168.211.132"
local port = 6379
red:connect(ip, port)
red:set("content_"..id, cjson.encode(res))
red:close()

ngx.say("{flag:true}")
```

