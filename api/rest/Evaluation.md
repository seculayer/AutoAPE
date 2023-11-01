# REST API - Evaluation

> Kubernetes Pod 내부 : *http://mrms-svc:9200* </br>
> Kubernetes Pod 외부 : *http://marster-ip:31920*


|Method|HTTP request|Description|
|------|------------|-----------|
|[**GetEvalResult**](Evaluation.md#GetEvalResult) | **GET** /mrms/get_eval_result |평가결과 반환|
|[**UpdateEvalResult**](Evaluation.md#UpdateEvalResult) | **POST** /mrms/eval_result_update |평가결과 DB 적용|


<a name="GetEvalResult"></a>
# **GetEvalResult**

### Parameters
```
?learn_hist_no=(String)
```

### Return Type
```
success : "1"
error : "error"
```

<a name="UpdateEvalResult"></a>
# **UpdateEvalResult**

### Parameters
```
Body: {"hist_no": (String),
       "task_idx": (String),
       "result": (String)}
```

### Return Type
```
success : "1"
error : "error"
```

