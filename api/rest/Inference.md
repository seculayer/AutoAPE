# REST API - Inference

> Kubernetes Pod 내부 : *http://mrms-svc:9200* </br>
> Kubernetes Pod 외부 : *http://marster-ip:31920*


|Method|HTTP request|Description|
|------|------------|-----------|
|[**UpdateInfrSttus**](Inference.md#UpdateInfrSttus) | **POST** /mrms/status_update/inference |검증 상태 업데이트|
|[**InferenceCreate**](Inference.md#InferenceCreate) | **GET** /mrms/inference_create |검증 생성|
|[**GetInferenceInfo**](Inference.md#GetInferenceInfo) | **GET** /mrms/get_inference_info |검증 상세 정보 반환(프로젝트id로 검색)|
|[**GetInferenceInfoDataset**](Inference.md#GetInferenceInfoDataset) | **GET** /mrms/get_inference_info_dataset |검증 상세 정보 반환(데이터셋id로 검색)|
|[**GetInfrSttusCd**](Inference.md#GetInfrSttudCd) | **POST** /mrms/get_infr_status_cd |검증 상태 업데이트|
|[**GetRunInference**](Inference.md#GetRunInference) | **GET** /mrms/get_run_inference |검증 상세 정보 반환(learn_hist_no로 검색)|
|[**UpdateInfrTime**](Inference.md#UpdateInfrTime) | **POST** /mrms/infr_time_update |검증 시작/종료 시각 업데이트|
|[**InferenceProgress**](Inference.md#InferenceProgress) | **GET** /mrms/inference_progress_rate |검증 진행률 반환|
|[**InferenceProgress**](Inference.md#InferenceProgress) | **POST** /mrms/inference_progress_rate |검증 진행률 업데이트|
|[**UpdateInfrResult**](Inference.md#UpdateInfrResult) | **GET** /mrms/update_infr_result |검증 결과 업데이트|


<a name="UpdateInfrSttus"></a>
# **UpdateInfrSttus**
현재 검증 상태를 실시간으로 해당 메소드 요청을 통해 업데이트
### Parameters
```
Body: {"hist_no": (String),
       "task_idx": (String),
       "status: (String),
       "message": (String)}
```
- **hist_no**
```
검증 Hist 번호(infr_hist_no)
```
- **task_idx**
```
이슈워커인덱스(issue_task_idx)
```
- **status** 
```
검증상태코드(infr_sttus_cd)
1(학습요청) / 2(학습대기) / 5(학습진행) / 6(학습완료) / 7(학습에러) 
```
- **message**
```
- 단순 상태 변경인 경우, message='-'
- 오류 발생인 경우, message = 오류 내용과 stack trace 내용 포함 
- 추후 생성 예정 
```
### Return Type
```
success : "1"
error : "error"
```

<a name="InferenceCreate"></a>
# **InferenceCreate**
학습이 완료된 모델과 검증에 사용할 데이터 셋을 선택하여 검증 생성
### Parameters
```
?learn_hist_no=(String)&data_analysis_id=(String)&target_field=(String)
```
- **learn_hist_no**
```
학습 Hist 번호
```
- **data_analysis_id**
```
데이터 분석 ID
```
- **target_field**
```
지정된 타겟 필드
```
### Return Type
```
success : "1"
error : "error"
```

<a name="GetInferenceInfo"></a>
# **GetInferenceInfo**
프로젝트 ID를 통해 수행한 검증에 대한 상세 정보 반환
### Parameters
```
?project_id=(String)
```
- **project_id**
```
프로젝트 ID
```
### Return Type
```
success : Json Format (String)
          {"infr_hist_no": (String),
           "infr_sttus_cd": (String),
           "xai_create_yn": (String),
           "start_time" : (String),
           "end_time": (String),
           "target_field": (String),
           ...}
error : "error"
```

<a name="GetInferenceInfoDataset"></a>
# **GetInferenceInfoDataset**
데이터셋 ID를 통해 수행한 검증에 대한 상세 정보 반환
### Parameters
```
?dataset_id=(String)
```
- **dataset_id**
```
데이터셋 ID
```
### Return Type
```
success : Json Format (String)
          {"infr_hist_no": (String),
           "infr_sttus_cd": (String),
           "xai_create_yn": (String),
           "start_time" : (String),
           "end_time": (String),
           "target_field": (String),
           ...}
error : "error"
```

<a name="GetInfrSttusCd"></a>
# **GetInfrSttusCd**
검증 HIST 번호를 통해 검증 상태(infr_sttus_cd)를 반환
### Parameters
```
Body: {"hist_no": (String)}
```
- **hist_no**
```
검증 Hist 번호(infr_hist_no)
```
### Return Type
```
success : status cd(String)
error : "error"
```

<a name="GetRunInference"></a>
# **GetRunInference**
학습 HIST 번호를 통해 수행한 검증에 대한 상세 정보 반환
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
success : List<Json> Format (String)
          {"infr_hist_no": (String),
           "infr_sttus_cd": (String),
           "start_time": (String),
           "end_time": (String),
           "target_field": (String)}
error : "error"
```

<a name="UpdateInfrTime"></a>
# **UpdateInfrTime**
검증 HIST 번호를 통해 검증 시작/종료 시각 업데이트
<br> type에 따라서 검증 시작(updateStartInfrTime)과 검증 종료(updateEndInfrTime) 각각 진행
### Parameters
```
Body: {"type": (String),
       "hist_no": (String)}
```
- **type**
```
"start" 또는 "end"
```
- **hist_no** 
``` 
검증 Hist 번호(infr_hist_no)
```
### Return Type
```
success : "1"
error : "error"
```

<a name="InferenceProgress"></a>
# **InferenceProgress**
검증 Hist 번호를 통해 검증 진행률 반환
### Parameters
```
?id=(String)
```
- **id**
```
검증 Hist 번호(infr_hist_no)
```
### Return Type
```
success : inference progress rate(String)
error : "error"
```

<a name="InferenceProgress"></a>
# **InferenceProgress**
검증이 시작될 때, "update" 모드로 전달받은 값을 통해 실시간으로 검증 진행률 정보를 업데이트
<br> 검증이 완료된 경우, "delete" 모드로 전달받은 값을 사용하여 해당 검증 HIST 번호에 대한 검증 진행률 정보를 제거
### Parameters
```
Body: {"infr_hist_no": (String),
       "progress_rate": (String),
       "mode": (String)}
```
- **infr_hist_no**
``` 
검증 Hist 번호(infr_hist_no)
```
- **progress_rate**
```
검증 진행률
```
- **mode**
```
update 또는 delete
```
### Return Type
```
success : "1"
error : "error"
```

<a name="UpdateInfrResult"></a>
# **UpdateInfrResult**
검증 HIST 번호에 따른 결과 값(검증 정확도, F1-Score) 업데이트
### Parameters
```
?infr_hist_no=(String)&infr_accuracy=(String)&infr_f1score=(String)
```
- **infr_hist_no** 
```
검증 Hist 번호(infr_hist_no)
```
- **infr_accuracy**
```
검증 정확도
```
- **infr_f1score**
```
검증 F1-Score
```
### Return Type
```
success : "1"
error : "error"
```

