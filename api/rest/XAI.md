# REST API - XAI

> Kubernetes Pod 내부 : *http://mrms-svc:9200* </br>
> Kubernetes Pod 외부 : *http://marster-ip:31920*


|Method|HTTP request|Description|
|------|------------|-----------|
|[**XAICreate**](XAI.md#XAICreate) | **GET** /mrms/xai_create |XAI 생성|
|[**UpdateXAICreateYN**](XAI.md#UpdateXAICreateYN) | **GET** /mrms/xai_create_yn_update |XAI 생성 이력 업데이트|
|[**XAIProgress**](XAI.md#XAIProgress) | **GET** /mrms/xai_progress_rate |XAI 진행률 반환|
|[**XAIProgress**](XAI.md#XAIProgress) | **POST** /mrms/xai_progress_rate |XAI 진행률 업데이트|
|[**UpdateXAISttus**](XAI.md#UpdateXAISttus) | **POST** /mrms/status_update/xai |XAI 상태 업데이트|
|[**GetXAIInfo**](XAI.md#GetXAIInfo) | **GET** /mrms/get_xai_info |XAI 정보 반환 <br> [{"xai_hist_no":(String), <br> "xai_sttus_cd":(String), <br> "alg_nm":(String), <br> "alg_cont":(String), <br> "learn_hist_no":(String)}]|


<a name="XAICreate"></a>
# **XAICreate**

### Parameters
```
?infr_hist_no=(String)&target_field=(String)&data_analysis_id=(String)&learn_hist_no=(String)
```

### Return Type
```
success : "1"
error : "error"
```

<a name="UpdateXAICreateYN"></a>
# **UpdateXAICreateYN**

### Parameters
```
?infr_hist_no=(String)
```

### Return Type
```
success : "1"
error : "error"
```

<a name="XAIProgress"></a>
# **XAIProgress**

### Parameters
```
?id=(String)
```

### Return Type
```
success : xai progress rate(String)
error : "error"
```

<a name="XAIProgress"></a>
# **XAIProgress**

### Parameters
```
Body: {"xai_hist_no": (String),
       "progress_rate": (String),
       "mode": (String)}
```

### Return Type
```
success : "1"
error : "error"
```

<a name="UpdateXAISttus"></a>
# **UpdateXAISttus**

### Parameters
```
Body: {"hist_no": (String),
       "sttus_cd": (String)}
```

### Return Type
```
success : "1"
error : "error"
```

<a name="GetXAIInfo"></a>
# **GetXAIInfo**

### Parameters
```
?project_id=(String)
```

### Return Type
```
success : Json Format(String)
error : "error"
```

