# REST API - Evaluation

> Kubernetes Pod 내부 : *http://mrms-svc:9200* </br>
> Kubernetes Pod 외부 : *http://marster-ip:31920*


|Method|HTTP request|Description|
|------|------------|-----------|
|[**GetEvalResult**](Evaluation.md#GetEvalResult) | **GET** /mrms/get_eval_result |평가결과 반환|
|[**UpdateEvalResult**](Evaluation.md#UpdateEvalResult) | **POST** /mrms/eval_result_update |평가결과 업데이트|


<a name="GetEvalResult"></a>
# **GetEvalResult**
학습 Hist 번호를 통해 평가 결과 반환(평과 결과 JSON)
### Parameters
```
?learn_hist_no=(String)
```
- **learn_hist_no**
```
학습 Hist 번호
```
### Return Type
```
success : "1"
error : "error"
```

<a name="UpdateEvalResult"></a>
# **UpdateEvalResult**
현재 평가 결과를 실시간으로 해당 메소드 요청을 통해 업데이트
### Parameters
```
Body: {"hist_no": (String),
       "task_idx": (String),
       "result": (String),
       "test_accuracy": (String)}
```
- **hist_no**
```
학습 Hist 번호(learn_hist_no)
```
- **task_idx**
```
이슈워커인덱스(issue_task_idx)
```
- **result**
```
평가결과 JSON(eval_result_json)
```
### Return Type
```
success : "1"
error : "error"
```

