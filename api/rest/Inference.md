# REST API - Inference

> Kubernetes Pod 내부 : *http://mrms-svc:9200* </br>
> Kubernetes Pod 외부 : *http://marster-ip:31920*


|Method|HTTP request|Description|
|------|------------|-----------|
|[**UpdateInfrSttus**](Inference.md#UpdateInfrSttus) | **POST** /mrms/status_update/inference |- 단순 상태 변경인 경우, message = '-' <br> - 오류 발생인 경우, message = 오류 내용과 stack trace 내용 포함 <br> - 추후 생성 예정|
|[**InferenceCreate**](Inference.md#InferenceCreate) | **GET** /mrms/inference_create |검증 생성|
|[**GetInferenceInfo**](Inference.md#GetInferenceInfo) | **GET** /mrms/get_inference_info |검증 정보 반환(프로젝트id로 검색) <br> {"infr_hist_no":(String), <br> "infr_sttus_cd":(String), <br> "xai_create_yn":(String), <br> "end_time":(String), <br> "target_field":(String), <br> ...}|
|[**GetInferenceInfoDataset**](Inference.md#GetInferenceInfoDataset) | **GET** /mrms/get_inference_info_dataset |검증 정보 반환(데이터셋id로 검색) <br> {"infr_hist_no":(String), <br> "infr_sttus_cd":(String), <br> "end_time":(String), <br> "target_field":(String), <br> ...}|
|[**GetInfrSttusCd**](Inference.md#GetInfrSttudCd) | **POST** /mrms/get_infr_status_cd |검증 상태 반환|
|[**GetRunInference**](Inference.md#GetRunInference) | **GET** /mrms/get_run_inference |검증 정보 반환(learn hist no로 검색) <br> {"infr_hist_no":(String), <br> "infr_sttus_cd":(String), <br> "end_time":(String), <br> "target_field":(String), <br> ...}|
|[**UpdateInfrTime**](Inference.md#UpdateInfrTime) | **POST** /mrms/infr_time_update |검증 시간 DB 적용|
|[**InferenceProgress**](Inference.md#InferenceProgress) | **GET** /mrms/inference_progress_rate |검증 진행률 반환|
|[**InferenceProgress**](Inference.md#InferenceProgress) | **POST** /mrms/inference_progress_rate |검증 진행률 업데이트|
|[**UpdateInfrResult**](Inference.md#UpdateInfrResult) | **GET** /mrms/update_infr_result |inferent 결과 업데이트|


<a name="UpdateInfrSttus"></a>
# **UpdateInfrSttus**

### Parameters
```
Body: {"hist_no": (String),
       "task_idx": (String),
       "status: (String),
       "message": (String)}
```

### Return Type
```
success : "1"
error : "error"
```

<a name="InferenceCreate"></a>
# **InferenceCreate**

### Parameters
```
?learn_hist_no=(String)&data_analysis_id=(String)&target_field=(String)
```

### Return Type
```
success : "1"
error : "error"
```

<a name="GetInferenceInfo"></a>
# **GetInferenceInfo**

### Parameters
```
?project_id=(String)
```

### Return Type
```
success : Json Format(String)
error : "error"
```

<a name="GetInferenceInfoDataset"></a>
# **GetInferenceInfoDataset**

### Parameters
```
?dataset_id=(String)
```

### Return Type
```
success : Json Format(String)
error : "error"
```

<a name="GetInfrSttusCd"></a>
# **GetInfrSttusCd**

### Parameters
```
Body: {"hist_no": (String)}
```

### Return Type
```
success : status cd(String)
error : "error"
```

<a name="GetRunInference"></a>
# **GetRunInference**

### Parameters
```
?learn_hist_no=(String)
```

### Return Type
```
success : List<Json> Format (String)
error : "error"
```

<a name="UpdateInfrTime"></a>
# **UpdateInfrTime**

### Parameters
```
Body: {"type": (String),
       "hist_no": (String)}
```

### Return Type
```
success : "1"
error : "error"
```

<a name="InferenceProgress"></a>
# **InferenceProgress**

### Parameters
```
?id=(String)
```

### Return Type
```
success : inference progress rate(String)
error : "error"
```

<a name="InferenceProgress"></a>
# **InferenceProgress**

### Parameters
```
Body: {"infr_hist_no": (String),
       "progress_rate": (String),
       "mode": (String)}
```

### Return Type
```
success : "1"
error : "error"
```

<a name="UpdateInfrResult"></a>
# **UpdateInfrResult**

### Parameters
```
?infr_hist_no=(String)&infr_accuracy=(String)&infr_f1score=(String)
```

### Return Type
```
success : "1"
error : "error"
```

