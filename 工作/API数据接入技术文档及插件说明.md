# 原理
模板可以看作是一群不同功能的节点的组合，为了实现一个功能（例如接口请求），需要不同的步骤。每个节点或者多个节点可以实现一个步骤，在`rule-ste`中定义和实现节点的功能
节点在模板和代码里面被表现为为`filter`

一个模板在运行时拥有一个独立的上下文，在代码里表现为`Context`类,核心的方法是`getVariable`和`setVariable`,即向上下文中添加或者获取变量.`Context`中维护了一个全局的`Map`,用来保存每个filter产生的结果和全局参数
对于每一个`filter`,需要继承抽象类`SteFilter`,这个抽象类实现了`Params`这个接口
这个接口定义了向上下文中添加参数和获得参数两个方法.对于`SteFilter`这个抽象类而言,它定义了所有的`filter`的行为,即每个`filter`都需要实现`process`,这个方法是`filter`的核心逻辑,主要的功能在这个方法里面实现

模板的执行主要依赖`SteDocumentProcess`这个类,这个类定义了几个模板执行的参数
其中`ResultBuilder`一般在模板最后定义,作用为选择结果写入的类和方法
在模板执行前,会先将`ResultBuilder`进行参数构建,因为模板处理数据的逻辑是边读边写
然后根据`SteFilter` 的 `processLine` 方法进行全模板Filter的初始化,将其添加到``SteDocumentProcess``的`builders`中,开始进行参数的初始化

在所有`filter`的参数都进行了初始化之后,开始按照顺序对所有的`filter`进行处理,全局有一个布尔类型的标记位为控制着是否停止模板的执行
为了保证循环处理能够正常进行,如果不是第一个`builder`,需要将其重置:重置意思是有循环处理的`filter`的状态进行初始化,即不再运行中和没有下一次循环
`filter`的循环意思是如果`filter`实现了`LoopFilter`接口,证明这个`filter`需要进行循环处理,核心方法是`hasNext`,`SteDocumentProcess`类根据这个方法进行判断
如果需要循环,就将这个`builder`对应的序号添加到栈中,

# 插件说明
## 模板全局常量及作用
### politeness
频率，秒为单位，0代表不限制
### proptype
生成文件名字的前缀，不添加默认为cube
### schemas
定义需要抓取的字段
### interval
定义的抓取间隔
### propname
模板名字，无实际意义
### useProxy
是否使用代理（需要在配置文件中配置代理服务器的ip）

# 执行流程

  

```Plain
// source.class:SourceBuilder
main(){    
    //内部处理流程
    ...code...
    f1();   
}

// process.0.class:ConfigProcessBuilder
function f1(main_arg...){
    // main_arg是main方法中定义的参数
    // f1内部处理流程
    // 对应filter.class所指定的插件
    filter1();
    filter2();
    // f1_arg是f1中定义的参数加上main传递过来的参数
    // 如果内部开启
    filterLoop(){
       fori
           filter3();
           f2(f1_arg);
    }
}

// process.1.class:ConfigProcessBuilder
function f2(f1_arg...){
    //其中f1_arg参数是f1中的传递下来的参数
    //对应filter.class所指定的插件
    filter21();
    filter22();
    //输出，全局定义的通用信息
    print();
}
```

  

# 方法定义

  

## 目前主要包含两种类型的方法

  

1. SourceBuilder 用于定义主方法，整理流程执行的入口。需要使用source.class:SourceBuilder定义。
    
2. ConfigProcessBuilder 子方法定义。需要使用process.0.class:ConfigProcessBuilder定义，其中0用于描述顺序，方便维护，无实际含义。
    

  

# 插件列表

  

## json类型插件

  

### JsonValueExtract：Json提取插件

```Plain
#提取token,eg:{"code":0,"message":"success","error_details":[],"request_id":"B34C8F27-F3CA-EE25-836C-32FABBD8B8CD","response_time":"2021-11-10 17:28:33","data":[{"sid":1,"mid":1,"name":"店铺1","seller_id":"**************","account_name":"account**","seller_account_id":2,"region":"EU","country":"西班牙"}]}
filter.class:JsonValueExtract
# 上文定义的来源信息
filter.parameters.from:body
# 要输出的结果，会放在上下文中
filter.parameters.to:shop_list
# 要从json中提取的key
filter.parameters.key:data
```

### JsonArraySizeExtract：提取JsonArray的大小

```Plain
filter.class:JsonArraySizeExtract
# 上文定义的来源信息
filter.parameters.from:shop_list
# 要输出的结果，会放在上下文中
filter.parameters.to:shop_count
```

### GenerateJsonFilter：生成嵌套Json的插件

### ToJsonFilter：转化Json插件

## 循环终止插件

### JsonArrayEmptyFilter：空数组跳出

```Plain
#判断是否有店铺数据，没有数据则忽略
filter.class:JsonArrayEmptyFilter
from:order_list  # 上文定义的参数信息
```

### HaveNextPageStopFilter：判断是否有下一页

```Plain
# 判断是否存在下一页数据，不存在则中断循环
filter.class:HaveNextPageStopFilter
# 总共要取的数量，可以直接定义或者动态从上文获取
total:total
# 偏移量。计算逻辑为：（num+limit）
offset:num
# 条数
limit:1000
```

  

## 循环插件

  

### JsonListMultiExtract：JsonArray循环抽取插件

```Plain
# jsonArrar的循环提取，该插件继承LoopFilter，会循环到所有数组中的信息全部提取完成。
filter.class:JsonListMultiExtract
# 上文定义的来源信息
from:shop_list
# 要从json中提取的key
to:shop_data
```

### LimitOffsetLoopFilter：按照偏移量分页插件

```Plain
# 从0开始获取偏移量
filter.class:LimitOffsetLoopFilter
# 条数
filter.parameters.limit:1000
# 初始偏移量
filter.parameters.offset:0
filter.parameters.maxValue:total //用于中断循环，如果没有指定最大值则会死循环。
to:num
```

### DateLoopFilter：日期循环插件

```Plain
# 时间迭代器，每过一次该过滤器，增加指定阈值，当to值达到规定的endDate值后，退出大的循环, 该插件继承LoopFilter，会循环到达到最大值为止。
filter.class:DateLoopFilter
# 指定的开始循环的时间节点，支持到秒。
beginDate:2022-11-01
# 结束时间，包含()的为默认支持的函数，目前支持4种类型的函数。
endDate:DAY_END()
# 增加的时间类型，目前支持6中类型的
## YEAR
## MONTH
## DAY
## HOUR
## MINUTE
## SECOND
incrTimeUnit:DAY
# 增加的时间，必须是数字
incrTimes:1
# 输出的开始时间点
toBeginDate:processBeginDate
# 输出的结束时间点
toEndDate:processEndDate
```

### CustomizeLoopFilter：自定义循环插件

```Plain
filter.class:CustomizeLoopFilter

## 循环终止条件
loopEnd
## 当前循环到的值 
current
## 循环最大长度
length
```

### MathLoopFilter

### MaxValueLimitFilter

### PageLoopFilter：分页插件

## 参数定义插件

### DefaultValueFilter：默认值插件

```Plain
//用于定义参数信息，以下示例定义了参数appId的值为ak_ERg93np9RWP9L
filter.class:DefaultValueFilter
appId:ak_ERg93np9RWP9L
```

  

## 日期处理插件

  

### DateAddFilter：用于处理时间的加减操作

```Plain
// 用于处理时间的加减操作
filter.class:DefaultValueFilter
// 时间来源定义，可以定义固定的时间或者从上文取。
from:2022-09-11/date_from
// 输出的结果
to:date_end
// 增量类型，YEAR:年，MONTH：月，DAY：日，HOUR：时，MINUTE：分，SECOND：秒
incrTimeUnit:YEAR/MONTH/DAY/HOUR/MINUTE/SECOND
// 增量
interval:1
```

### TimeConvertFilter：时间转化插件

```Plain
# 时间转化插件
filter.class:TimeConvertFilter
# 输入
filter.parameters.from:processBeginDate
# 输出
filter.parameters.to:toBeginDate
# 格式化信息
filter.parameters.format:yyyy-MM-dd
```

## http插件

  

### UrlRequestFilter：http请求插件

```Plain
filter.class:UrlRequestFilter
url:page_url 要请求的url,支持动态传入。
to:order_body 请求成功后放入上文中的信息
post:true 是否post的标识，默认为false
headers.Content-Type:application/json: header 设置，如果post参数类型为raw,必须设置为：application/json
params.start_date:toBeginDate 参数，支持动态传入
params.end_date:toEndDate
params.date_type:2
params.offset:num
params.sid:sid
params.length:1000

#get请求需要
needEncode:true
```

### HttpStatusCheckFilter：http状态码校验

```Plain
filter.class:HttpStatusCheckFilter
checkCode:code 上文中定义的参数
successCode:0 自己定义的值信息
errorMsg:请求异常/shop_body ,可以设置固定的信息或者动态参数从上文获取。
failFast:true  //是否快速失败，默认是，如果设置成false,失败后会跳过流程，继续后续执行。
```

## 字符串处理插件

  

### BindStringFilter：用于字符串的动态拼接的处理

```Plain
# 用于字符串的动态拼接的处理，可以定义分隔符，用"扩起来的无需要替换，没有引号括起来的会从上文中动态去获取
filter.class:BindStringFilter
# 要处理的字符串信息
attrs:"https://inara.yucekj.com/lingxing/erp/sc/data/seller/lists?access_token=" accessToken "&app_key=" appId "&timestamp=" requestTime "&sign=" sign
# 输出到上文中的字符串信息
to:page_url
# spliter：分割符，可以自定义，默认按照空格分割
```

  

## 领星扩展插件

### LingXingTokenFilter

```Plain
# 领星获取token，领星token需要的签名信息从上文中定义的appId，appSecret去获取。由于领星的token获取不能重复获取，重复获取会导致原有的token实效，所有该功能属于定制化（缓存）
filter.class:LingXingTokenFilter
# 获取授权token的地址信息
url:https://inara.yucekj.com/lingxing/api/auth-server/oauth/access-token
# 输出token信息到上下文中
to:accessToken
```

  

### LingXingSignFilter

```Plain
# 请求签名
filter.class:LingXingSignFilter
# 签名的时间，签名后返回的内容
requestTime:requestTime
# 签名信息，签名后返回的内容
sign:sign
dynamicParameter.end_date:toEndDate
params.date_type:2
# dynamicParameter 开头的代表动态信息，会从上下文中获取。eg:dynamicParameter.end_date:toEndDate，
# params开头的为普通参数，直接按照传入的处理。eg:params.date_type:2
```

  

## 平台化功能

使用该功能，需要在脚本中定义出平台类型（platform:KUAISHOU）。新的saas上线后不需要额外定义。

### TokenFilter

```Plain
# 统一获取token服务（后续替换所有签名）
filter.class:TokenFilter
to:accessToken
```

### SignFilter

```Plain
# 统一签名服务（后续替换所有签名）
filter.class:SignFilter
params.method:open.user.info.get
params.version:1
```

  

## ToArrayFilter：转数组插件

```Plain
filter.class:ToArrayFilter
# 要转换的值
from:
# 转换后的值
to:
# 格式，转换之前的格式
# "1,2,3,4,5,6"
# "a,b,c,d,e,f,g"
format:String或者Integer
```

## OSS插件

  

```Plain
filter.class:OssFileContentReadFilter

## 最终输出的值
to

## 对应OSS路径
path

## 字符集
charset
```

  

## 抽取插件

### CombineExtractFilter

### HrefExtractFilter

### CssExtractFilter

### MultiExtract

### MultiGetFilterFromFile

### PatternExtractFilter

### PatternMultiExtractFilter

### SeqExtractFilter

## 其他

### RandomCharFilter：随机字符串插件

### PrintFilter：输出日志插件

  

# 函数

## 支持的函数

1. CURRENT_DATE()，获取当前时间yyyy-MM-dd
    
2. CURRENT_TIMESTAMP()，当前时间yyyy-MM-dd HH:mm:ss
    
3. MORNING() 当日最早时间yyyy-MM-dd HH:mm:ss,eg:2022-09-11 00:00:00
    
4. DAY_END() 当日最晚时间yyyy-MM-dd HH:mm:ss,eg:2022-09-11 23:59:59
    

  

## 函数使用

```Plain
filter.class:DateLoopFilter
# 值中包含()的为函数，目前支持4种类型的函数。
endDate:DAY_END()
```