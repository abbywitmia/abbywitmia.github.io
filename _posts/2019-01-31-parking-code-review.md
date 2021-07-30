---
layout: post
title: Parking Code Review
categories: Java
description: Parking Code Review
keywords: XXL-JOB, Java, 分片广播
---

# 跬步1 Parking Code Review Demo

程序员有三大美德
* 懒惰(Laziness)
* 急躁(Impatience)
* 傲慢(Hubris)

quoted from Larry Wall

20190131 Parking team进行了code review  
记录一下自己的一些感想,欢迎拍砖
## 1 Builder
### 1.1 Code
```java
/**
 * 构建ETCPBaseRequestVo
 * @param object
 * @return ETCPBaseRequestVo
 * @throws JsonProcessingException
 */
public ETCPBaseRequestVo buildETCPBaseRequestVo(Object object) throws JsonProcessingException{
    String result = JsonParseUtil.objectToJSON(object);
    String timestamp = DateUtil.getYYYYMMddHHmmss();
    String singedData = DataEncryptionUtil.getSignedData(result, timestamp, merchantConfig.getKey());
    ETCPBaseRequestVo etcpBaseRequestVo = new ETCPBaseRequestVo();
    etcpBaseRequestVo.setMerchant_no(merchantConfig.getNumber());
    etcpBaseRequestVo.setTime_stamp(timestamp);
    etcpBaseRequestVo.setSign(singedData);
    etcpBaseRequestVo.setData(result);
    return etcpBaseRequestVo;
}
```
### 1.2 What’s the point
当我们构造一个类的实例的时候,拢共分几步?  
三步,和把大象装冰箱一样哎!  
1调用构造函数(打开冰箱门)
```java
ETCPBaseRequestVo etcpBaseRequestVo = new ETCPBaseRequestVo();
```
2一个个set参数(把大象放进去)
```java
etcpBaseRequestVo.setTime_stamp(timestamp);
etcpBaseRequestVo.setSign(singedData);
etcpBaseRequestVo.setData(result);
```
3return(把冰箱门带上)
```java
return etcpBaseRequestVo;
```
懒惰的我发现第二步,我要写好多遍实例名`etcpBaseRequestVo`,好烦!  
怎么能少写点code?  
低糖的Java加糖用Lombok.
### 1.3 Demo
```java
return etcpBaseRequestVo.Builder()
  .time_stamp(timestamp)
  .sign(singedData);
  .data(result)
  .build();
```
POJO类加上`@Builder`注解就可以如上使用了
```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@Accessors(chain = true)
@Builder
public class ETCPBaseRequestVo implements Serializable {
    /**
     * 商户号
     */
    private String merchant_no;
    /**
     * 时间戳
     */
    private String time_stamp;
    /**
     * 签名
     */
    private String sign;
    /**
     * 业务数据参数
     */
    private String data;
}
```
## 2 If and Else
### 2.1 Code
```java
/**
 * 请求 ETCP 获取Token
 * @param mobilePhoneNo
 * @return String
 * @throws JsonProcessingException
 */
public String requestUsersigin(String mobilePhoneNo) throws JsonProcessingException{
    String token = "";
    //联合登录获取Token Vo
    UsersiginRequestVo usersiginRequestVo = new UsersiginRequestVo();
    usersiginRequestVo.setAppId(merchantConfig.getAppId());
    usersiginRequestVo.setMobilePhone(mobilePhoneNo);
    //构建ETCPBaseRequestVo
    ETCPBaseRequestVo usersiginBaseRequestVo = buildETCPBaseRequestVo(usersiginRequestVo);
    //call ETCP API[usersigin]
    UsersiginResponseVo usersiginResponseVo = etcpFeignClient.userSignIn(usersiginBaseRequestVo);
    if(null != usersiginResponseVo && null != usersiginResponseVo.getCode()){
        if(usersiginResponseVo.getCode().equals(ETCPBusinessCodeEnum.SUCCESS.getCode())){
            token = usersiginResponseVo.getData().getToken();
        }else{
            log.error(usersiginResponseVo.getCode() + ":" + usersiginResponseVo.getMessage());
        }
    }else{
        log.error("call ETCP API[usersigin] response is null");
    }
    return token;
}
```
### 2.2 What’s the point
当我看到一堆if嵌套引起的缩进的时候,我是懵逼的(不知道游标卡尺master python程序员怎么想).  
其实这个还好,只有两层,不过当分支block过大或嵌套更深时,仍然非常可怕  
因为缩进某种意义是个栈结构,我理解大多数人的栈深度并不大  
我承认我脑子转的不够快,但能不能写一些对我这样的迟钝者友好些的代码呢?  
答案是可以,提前return就好了  
如果有一个if/else,处理逻辑一个长一个短,把短的提前判断出来,然后return
### 2.3 Demo
```java
/**
 * 请求 ETCP 获取Token
 * @param mobilePhoneNo
 * @return String
 * @throws JsonProcessingException
 */
public String requestUsersigin(String mobilePhoneNo) throws JsonProcessingException{
    String token = "";
    //联合登录获取Token Vo
    UsersiginRequestVo usersiginRequestVo = new UsersiginRequestVo();
    usersiginRequestVo.setAppId(merchantConfig.getAppId());
    usersiginRequestVo.setMobilePhone(mobilePhoneNo);
    //构建ETCPBaseRequestVo
    ETCPBaseRequestVo usersiginBaseRequestVo = buildETCPBaseRequestVo(usersiginRequestVo);
    //call ETCP API[usersigin]
    UsersiginResponseVo usersiginResponseVo = etcpFeignClient.userSignIn(usersiginBaseRequestVo);
    if(Objects.isNull(usersiginResponseVo) || Objects.isNull(usersiginResponseVo.getCode())) {
      log.error("call ETCP API[usersigin] response is null");
      return token;
    }
    if(usersiginResponseVo.getCode().equals(ETCPBusinessCodeEnum.SUCCESS.getCode())){
        return usersiginResponseVo.getData().getToken();
    }
    log.error(usersiginResponseVo.getCode() + ":" + usersiginResponseVo.getMessage());
    return token;
}
```
## 3 Equals
### 3.1 Code
```java
/**
 * 请求 ETCP 获取Token
 * @param mobilePhoneNo
 * @return String
 * @throws JsonProcessingException
 */
public String requestUsersigin(String mobilePhoneNo) throws JsonProcessingException{
    String token = "";
    //联合登录获取Token Vo
    UsersiginRequestVo usersiginRequestVo = new UsersiginRequestVo();
    usersiginRequestVo.setAppId(merchantConfig.getAppId());
    usersiginRequestVo.setMobilePhone(mobilePhoneNo);
    //构建ETCPBaseRequestVo
    ETCPBaseRequestVo usersiginBaseRequestVo = buildETCPBaseRequestVo(usersiginRequestVo);
    //call ETCP API[usersigin]
    UsersiginResponseVo usersiginResponseVo = etcpFeignClient.userSignIn(usersiginBaseRequestVo);
    if(null != usersiginResponseVo && null != usersiginResponseVo.getCode()){
        if(usersiginResponseVo.getCode().equals(ETCPBusinessCodeEnum.SUCCESS.getCode())){
            token = usersiginResponseVo.getData().getToken();
        }else{
            log.error(usersiginResponseVo.getCode() + ":" + usersiginResponseVo.getMessage());
        }
    }else{
        log.error("call ETCP API[usersigin] response is null");
    }
    return token;
}
```
### 3.2 What’s the point
```java
if(usersiginResponseVo.getCode().equals(ETCPBusinessCodeEnum.SUCCESS.getCode()))
```
这段equals是可能抛空指针的  
如果你装了alibaba的编码规约插件,你可能会看到warning,让你把非空变量作为equals函数调用方  
另一种办法是使用java built-in的`Objects.equals()`
### 3.3 Demo
```java
if(Objects.equals(usersiginResponseVo.getCode(), ETCPBusinessCodeEnum.SUCCESS.getCode())
```
## 4 Let it crash
### 4.1 Code
```java
/**
 * 请求 ETCP 获取Token
 * @param mobilePhoneNo
 * @return String
 * @throws JsonProcessingException
 */
public String requestUsersigin(String mobilePhoneNo) throws JsonProcessingException{
    String token = "";
    //联合登录获取Token Vo
    UsersiginRequestVo usersiginRequestVo = new UsersiginRequestVo();
    usersiginRequestVo.setAppId(merchantConfig.getAppId());
    usersiginRequestVo.setMobilePhone(mobilePhoneNo);
    //构建ETCPBaseRequestVo
    ETCPBaseRequestVo usersiginBaseRequestVo = buildETCPBaseRequestVo(usersiginRequestVo);
    //call ETCP API[usersigin]
    UsersiginResponseVo usersiginResponseVo = etcpFeignClient.userSignIn(usersiginBaseRequestVo);
    if(null != usersiginResponseVo && null != usersiginResponseVo.getCode()){
        if(usersiginResponseVo.getCode().equals(ETCPBusinessCodeEnum.SUCCESS.getCode())){
            token = usersiginResponseVo.getData().getToken();
        }else{
            log.error(usersiginResponseVo.getCode() + ":" + usersiginResponseVo.getMessage());
        }
    }else{
        log.error("call ETCP API[usersigin] response is null");
    }
    return token;
}
```
### 4.2 What’s the point
这段在遇到异常的情况下返回是空字符串""  
但是如果token是空的情况,业务肯定失败,这时应该直接抛出`Exception`  
有问题请直接失败,这就是失败策略之fail-fast
### 4.3 Demo
```java
public String requestUsersigin(String mobilePhoneNo) throws JsonProcessingException{
    String token;
    //联合登录获取Token Vo
    UsersiginRequestVo usersiginRequestVo = new UsersiginRequestVo();
    usersiginRequestVo.setAppId(merchantConfig.getAppId());
    usersiginRequestVo.setMobilePhone(mobilePhoneNo);
    //构建ETCPBaseRequestVo
    ETCPBaseRequestVo usersiginBaseRequestVo = buildETCPBaseRequestVo(usersiginRequestVo);
    //call ETCP API[usersigin]
    UsersiginResponseVo usersiginResponseVo = etcpFeignClient.userSignIn(usersiginBaseRequestVo);
    if(Objects.isNull(usersiginResponseVo) || Objects.isNull(usersiginResponseVo.getCode())) {
      throw new RuntimeException("call ETCP API[usersigin] response is null");
    }
    if(usersiginResponseVo.getCode().equals(ETCPBusinessCodeEnum.SUCCESS.getCode())){
        return usersiginResponseVo.getData().getToken();
    }
    throw new RuntimeException(usersiginResponseVo.getCode() + ":" + usersiginResponseVo.getMessage());
}
```
## 5 Expire token
### 5.1 Code
```java
/**
 * 绑定车牌
 * @param vehiclePlateInfoRequestVo
 * @return
 */
public ServiceResponseVo bindVehiclePlate(VehiclePlateInfoRequestVo vehiclePlateInfoRequestVo) throws JsonProcessingException {
    //ServiceResponseVo serviceResponseVo = null;
    //获取手机号
    String mobilePhoneNo = getMobilePhoneNo(vehiclePlateInfoRequestVo.getUserId());
    //获取用户Id
    String userId = vehiclePlateInfoRequestVo.getUserId();
    //获取token
    String token = getToken(mobilePhoneNo, userId);
    //绑定车牌请求Vo
    BindcarRequestVo bindcarRequestVo = new BindcarRequestVo();
    bindcarRequestVo.setToken(token);
    bindcarRequestVo.setPlateNumber(vehiclePlateInfoRequestVo.getPlateNo());
    //call ETCP API[bindcar]
    BindcarResponseVo bindcarResponseVo = requestBindVehiclePlate(bindcarRequestVo);
    //响应包处理
    ServiceResponseVo serviceResponseVo = bindVehiclePlateResponseHandle(bindcarResponseVo);
    //token 失效
    if(serviceResponseVo.getRespCode().equals(ETCPBusinessCodeEnum.TOKEN_INVALID.getCode())){
        //call ETCP API[bindcar]
        bindcarResponseVo = requestBindVehiclePlate(bindcarRequestVo);
        //响应包处理
        serviceResponseVo = bindVehiclePlateResponseHandle(bindcarResponseVo);
    }
    return serviceResponseVo;
}
```
### 5.2 What’s the point
这是绑定车牌的逻辑,需要从缓存获取token  
token可能过期,过期时应该更新token  
这个实现有一个问题就是没有在刷新token时缓存失效  
可能会导致业务在token过期的间隔中持续失败
### 5.3 Demo
这个是payment的充电接入层实现
```java
@SneakyThrows
public <Req, Resp> Resp call(String method, Req data, Class<Resp> clazz) {
    return call(method, data, clazz, getToken());
}

private <Req, Resp> Resp call(@NonNull String method, @NonNull Req data, @NonNull Class<Resp> clazz, String token) throws CommonException {
    String url = properties.getBaseUrl() + method;
    CommonResponse response = restTemplate.postForObject(url, new HttpEntity<>(getRequestDatagram(data), getHeaders(token)), CommonResponse.class);
    if (Objects.isNull(response)) {throw new CommonException(CommonErrorEnum.DOWNSTREAM_SYSTEM_ERROR);}
    CommonErrorEnum errorEnum = EnumUtil.map(response.getCode(), EvcsErrorEnum.class, null, CommonErrorEnum.class, CommonErrorEnum.DOWNSTREAM_SYSTEM_ERROR, true);
    if (Objects.equals(CommonErrorEnum.TOKEN_INVALID, errorEnum)) {
        redisClient.delete(properties.getTokenKey());
        return call(method, data, clazz);
    }
    if (Objects.equals(CommonErrorEnum.SUCCESS, errorEnum)) {
        byte[] resp = decrypt(response.getData());
        try {
            return JsonUtil.readValue(resp, clazz);
        } catch (Exception e) {
            log.error("can not deserialize to {}, response data is {{}}, request is {{}}", clazz.getSimpleName(), new String(resp), data);
            throw e;
        }
    }
    log.error("evcs response error: {{}}, and request is {}, url is {}", response, JsonUtil.writeValueAsString(data), url);
    throw new CommonException(errorEnum, StringUtils.defaultIfBlank(response.getMessage(), errorEnum.getShowMessage()));
}

public String getToken() throws CommonException {
    String token = redisClient.get(properties.getTokenKey());
    if (StringUtils.isNotBlank(token)) {return token;}
    TokenResponse tokenResponse = call(EvcsProperties.TOKEN_API, new TokenRequest(properties.getLocalOperatorId(), properties.getServerOperatorSecret()), TokenResponse.class, null);
    if (!EvcsErrorEnum.SUCCESS.getValue().equals(tokenResponse.getErrorMessage())) {throw new CommonException(CommonErrorEnum.DOWNSTREAM_SYSTEM_ERROR);}
    redisClient.put(properties.getTokenKey(), tokenResponse.getToken(), tokenResponse.getAvailableTime(), TimeUnit.SECONDS);
    return tokenResponse.getToken();
}
```
## 6 Why not use JsonAlias
### 6.1 Code
```java
/**
 * 查询在场车辆的停车费用 响应包处理
 * @param orderunpayResponseVo
 * @return List<ParkFeeInfoResponseVo>
 */
public List<ParkFeeInfoResponseVo> unPayParkFeeInfoResponseHandle(OrderunpayResponseVo orderunpayResponseVo) throws ParseException{
    List<ParkFeeInfoResponseVo> parkFeeInfoResponseVoList= new ArrayList<ParkFeeInfoResponseVo>();
    if(null != orderunpayResponseVo && null != orderunpayResponseVo.getCode()){
        if(orderunpayResponseVo.getCode() == 0){
            OrderunpayDataVo [] orderunpayDataVo = orderunpayResponseVo.getData();
            for (OrderunpayDataVo orderunpayDataVoTemp : orderunpayDataVo) {
                ParkFeeInfoResponseVo parkFeeInfoResponseVo = new ParkFeeInfoResponseVo();
                parkFeeInfoResponseVo.setOrderId(orderunpayDataVoTemp.getOrderId());
                parkFeeInfoResponseVo.setEntranceTime(orderunpayDataVoTemp.getEntranceTime());
                Long parkingTime = DateUtil.getDiffTime(orderunpayDataVoTemp.getEntranceTime(),orderunpayDataVoTemp.getExitTime());
                parkFeeInfoResponseVo.setParkingTime(parkingTime.toString());
                parkFeeInfoResponseVo.setTotalParkingFee(orderunpayDataVoTemp.getParkingFee());
                parkFeeInfoResponseVo.setPaidParkingFee(orderunpayDataVoTemp.getPaidFee());
                parkFeeInfoResponseVo.setRemainingParkingFee(orderunpayDataVoTemp.getRemainingParkFee());
                parkFeeInfoResponseVo.setRemainingFee(orderunpayDataVoTemp.getRemainingFee());
                parkFeeInfoResponseVo.setPaidServiceFee(orderunpayDataVoTemp.getPaidServiceFee());
                parkFeeInfoResponseVo.setRemainingServiceFee(orderunpayDataVoTemp.getRemainingServiceFee());
                parkFeeInfoResponseVo.setCoupon(orderunpayDataVoTemp.getDiscountAmount().toString());
                parkFeeInfoResponseVo.setPlateNo(orderunpayDataVoTemp.getCarNumber());
                parkFeeInfoResponseVo.setParkId(orderunpayDataVoTemp.getParkingId());
                parkFeeInfoResponseVo.setParkName(orderunpayDataVoTemp.getParkingName());
                parkFeeInfoResponseVoList.add(parkFeeInfoResponseVo);
            }
        }else if(orderunpayResponseVo.getCode().equals(ETCPBusinessCodeEnum.TOKEN_INVALID.getCode())){
            log.error(orderunpayResponseVo.getCode() + ":" + orderunpayResponseVo.getMessage());
        }else{
            log.error(orderunpayResponseVo.getCode() + ":" + orderunpayResponseVo.getMessage());
        }
    }else{
        log.error("call ETCP API[orderunpay] response is null");
    }
    return parkFeeInfoResponseVoList;
}
```
### 6.2 What’s the point
这段的逻辑是把etcp的返回结果包装成我们自己的内部逻辑处理类  
那么问题来了,为啥不直接用内部逻辑类作为etcp的返回结果?  
这里有一个问题,两边的字段名不一致怎么办?  
可能有人会说,内部类的命名是我们可以控制的,我们匹配etcp的命名不就好了?  
那么如果停简单的字段名和etcp不一致\(事实上真的不一致,再退一步,如果以后接别的CP/SP怎么办)怎么办?  
我们需要更通用的方法
### 6.3 Demo
```java
public class InternalOrder {
  @JsonAlias({"OrderId", "order_id"})
  String orderId;
}
```
这样不论是
```json
{"OrderId":"testOrderId"}
```
还是
```json
{"order_id":"testOrderId"}
```
都可以反序列化成功  
By the way 这个例子嵌套导致的缩进同样很多,可以用前述方式[2 If and Else](#2-if-and-else)来实现
## 7 Global excpetion handler
### 7.1 Code
```java
/**
 * 绑定车牌
 * @param vehiclePlateInfoRequestVo
 * @return MAResponse<VoidVo>
 */
@PostMapping(value = "/etcp/bindvehicleplate")
public MAResponse<VoidVo> bindVehiclePlate(@RequestBody VehiclePlateInfoRequestVo vehiclePlateInfoRequestVo){
    MAResponse<VoidVo> maResponse = null;
    try{
        log.info("call /etcp/bindvehicleplate begin");
        ServiceResponseVo serviceResponseVo = etcpAdaptService.bindVehiclePlate(vehiclePlateInfoRequestVo);
        maResponse = MAResponse.buildMAResponse(serviceResponseVo.getRespCode(), serviceResponseVo.getRespMsg());
        log.info("call /etcp/bindvehicleplate end");
    } catch (Exception e){
        log.error("call /etcp/bindvehicleplate error", e);
        maResponse = MAResponse.buildMAResponse(ServiceResponseType.ERROR.getRespCode(), ServiceResponseType.ERROR.getMessage());
    }
    return maResponse;
}
```
### 7.2 What’s the point
每个操作都要try/catch和打日志,好烦  
可以用注解来打日志,用global excpetion handler来处理异常
### 7.3 Demo
```java
/**
 * 绑定车牌
 * @param vehiclePlateInfoRequestVo
 * @return MAResponse<VoidVo>
 */
@Stalker
@PostMapping(value = "/etcp/bindvehicleplate")
public MAResponse<VoidVo> bindVehiclePlate(@RequestBody VehiclePlateInfoRequestVo vehiclePlateInfoRequestVo){
    MAResponse<VoidVo> maResponse = null;
    log.info("call /etcp/bindvehicleplate begin");
    ServiceResponseVo serviceResponseVo = etcpAdaptService.bindVehiclePlate(vehiclePlateInfoRequestVo);
    maResponse = MAResponse.buildMAResponse(serviceResponseVo.getRespCode(), serviceResponseVo.getRespMsg());
    log.info("call /etcp/bindvehicleplate end");
    return maResponse;
}
```
环绕日志注解,包括出入参和运行时间
```java
@Aspect
@Component
@Slf4j
public class AspectConfig {

    @Around("@annotation(com.ezia.coreservices.common.annotation.Stalker)")
    public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
        try {
            long startTime = System.nanoTime();
            Object result = joinPoint.proceed();
            long costTime = System.nanoTime() - startTime;

            log.info("class {} method {} costs {} with request {{}} and result {{}}",
                    joinPoint.getTarget().getClass().getSimpleName(),
                    joinPoint.getSignature().getName(),
                    costTime,
                    Arrays.toString(joinPoint.getArgs()),
                    result);

            return result;
        } catch (Exception e) {
            log.info("class {} method {} error with request {{}}",
                    joinPoint.getTarget().getClass().getSimpleName(),
                    joinPoint.getSignature().getName(),
                    Arrays.toString(joinPoint.getArgs()),
                    e);
            throw e;
        }
    }
}
```
全局异常handler
```java
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

    private static ApiResult<?> handle(Exception e, ErrorEnum errorEnum) {
        return handle(e, errorEnum, null);
    }

    private static ApiResult<?> handle(Exception e, ErrorEnum errorEnum, String errMsg) {
        log.error(e.getMessage(), e);
        return ApiResult.error(errorEnum, errMsg);
    }

    @ExceptionHandler(BindException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ApiResult<?> bindException(BindException e) {
        return handle(e, PARAM_INVALID,
                e.getBindingResult().getFieldErrors().stream()
                        .map(DefaultMessageSourceResolvable::getDefaultMessage)
                        .findFirst().orElse(PARAM_INVALID.getShowMessage()));
    }

    @ExceptionHandler(ConstraintViolationException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ApiResult<?> constraintViolationException(ConstraintViolationException e) {
        return handle(e, PARAM_INVALID, e.getMessage());
    }

    @ExceptionHandler(MissingServletRequestParameterException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ApiResult<?> missingServletRequestParameterException(Exception e) {
        return handle(e, CommonErrorEnum.PARAM_ABSENT, e.getMessage());
    }

    @ExceptionHandler(HttpRequestMethodNotSupportedException.class)
    @ResponseStatus(HttpStatus.METHOD_NOT_ALLOWED)
    public ApiResult<?> httpRequestMethodNotSupportedException(HttpRequestMethodNotSupportedException e) { return handle(e, CommonErrorEnum.HTTP_REQUEST_METHOD_INVALID); }

    @ExceptionHandler(CommonException.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ApiResult<?> commonException(CommonException e) { return handle(e, e.getErrorEnum()); }

    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ApiResult<?> internalServerError(Exception e) { return handle(e, CommonErrorEnum.UNKNOWN_ERROR); }
}
```
