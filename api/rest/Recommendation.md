# REST API - Recommendation

> Kubernetes Pod 내부 : *http://mrms-svc:9200* </br>
> Kubernetes Pod 외부 : *http://marster-ip:31920*


|Method|HTTP request|Description|
|------|------------|-----------|
|[**GetVarFuncList**](Recommendation.md#GetVarFuncList) | **GET** /mrms/get_cvt_fn |DB에 저장된 데이터변환 함수 리스트 반환|
|[**InsertDPAnalsInfo**](Recommendation.md#InsertDPAnalsInfo) | **POST** /mrms/insert_dp_anls_info |dprs 결과 저장|
|[**InsertAlgAnlsInfo**](Recommendation.md#InsertAlgAnlsInfo) | **POST** /mrms/insert_alg_anls_info |mars 결과 저장|
|[**GetAlgorithmInfo**](Recommendation.md#GetAlgorithmInfo) | **GET** /mrms/get_algorithm_info |algorithm 정보 반환|
|[**GetParamInfo**](Recommendation.md#GetParamInfo) | **GET** /mrms/get_param_info |특정 알고리즘에 대한 hyper parameter 구성 정보 반환|
|[**InsertMLParamInfo**](Recommendation.md#InsertMLParamInfo) | **POST** /mrms/insert_ml_param_info |프로젝트에서 사용될 파라미터 선택 후 추가|


<a name="GetVarFuncList"></a>
# **GetVarFuncList**

### Parameters
```
-
```

### Return Type
```
success : List<Json> Format (String)
          ex) [{"conv_func_cls":"NotNormal","conv_func_tag":"[[@not_normal()]]"}]
error : "error"
```

<a name="InsertDPAnalsInfo"></a>
# **InsertDPAnalsInfo**

### Parameters
```
Body: [{"data_analysis_json": (String),
        "data_analysis_id": (String)},
        ...
      ]
```

### Return Type
```
success : "1"
error : "error"
```

<a name="InsertAlgAnlsInfo"></a>
# **InsertAlgAnlsInfo**

### Parameters
```
Body: [{"alg_id": (String),
        "metadata_json": (String),
        "alg_type": (String),
        "alg_json": (String)},
        ...
      ]
```

### Return Type
```
success : "1"
error : "error"
```

<a name="GetAlgorithmInfo"></a>
# **GetAlgorithmInfo**

### Parameters
```
-
```

### Return Type
```
success : "1"
error : "error"
```

<a name="GetParamInfo"></a>
# **GetParamInfo**

### Parameters
```
?alg_id=(String)
```

### Return Type
```
success : "1"
error : "error"
```

<a name="InsertMLParamInfo"></a>
# **InsertMLParamInfo**

### Parameters
```
Body: {"learn_alg_id": (String),
       "data_process_id": (String),
       "project_id": (String),
       "param_json": (String)}
```

### Return Type
```
success : "1"
error : "error"
```