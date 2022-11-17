# REST API - Management
> 추후 업데이트 예정입니다.

> Kubernetes Pod 내부 : *http://mrms-svc:9200* </br>
> Kubernetes Pod 외부 : *http://marster-ip:31920*

  
|Method|HTTP request|Description|
|------|------------|-----------|
|[**getConverterFunctions**](Managements.md#getConverterFunctions) | **GET** /mrms/get_cvt_fn |모든 데이터변환 함수 정보 리스트 반환|
|[**getAlgorithmInfo**](Managements.md#getAlgorithmInfo) | **GET** /get_algorithm_info |모든 ML/DL 알고리즘 정보 리스트 반환|


<a name="getConverterFunctions"></a>
# **getConverterFunctions**

### Parameters
This endpoint does not need any parameter.

### Return Type
Return type is string that converted list of JSON format.
```json
[{"conv_func_cls":"NotNormal","conv_func_tag":"[[@not_normal()]]"}]
```
<a name="getAlgorithmInfo"></a>
# **getAlgorithmInfo**

### Parameters
This endpoint does not need any parameter.

### Return Type
Return type is string that converted list of JSON format.
```json
[{"alg_id":"20000000000000001","alg_nm":"K-DNN","alg_type":"1,2"}]
```
