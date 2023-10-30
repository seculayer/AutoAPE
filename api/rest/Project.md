# REST API - Project

> Kubernetes Pod 내부 : *http://mrms-svc:9200* </br>
> Kubernetes Pod 외부 : *http://marster-ip:31920*


|Method|HTTP request|Description|
|------|------------|-----------|
|[**ProjectCreate**](Project.md#ProjectCreate) | **POST** /mrms/project_create |- 신규 Project DB 추가 요청 <br> - modeling_mode <br>&nbsp; 1: 기본, 2: 재학습 <br> - early_stop_param <br>&nbsp; {"early_type" : 0,1,2,3 (string), # 0: None, 1: min, 2: max, 3: variance <br>&nbsp; "early_key" : (string), # "accuracy" or "loss" or "val_loss" or "val_accuracy" <br>&nbsp; "early_value" : (float), # 0에서 1 사이의 수 <br>&nbsp; "minsteps" : (int) # 최소 스텝 수} <br> - tag <br>&nbsp; comma(",") split string|
|[**GetProjectSttusCd**](Project.md#GetProjectSttusCd) | **GET**  /mrms/get_proj_sttus_cd |해당 project_id 현재 상태를 반환| 
|[**GetModelSttusCd**](Project.md#GetModelSttusCd) | **POST** /mrms/get_sttus_cd |해당 hist_no의 학습 상태를 반환(Project status와는 별개)|
|[**UpdateModelSttus**](Project.md#UpdateModelSttus) | **POST** /mrms/status_update/learn |- 단순 상태 변경인 경우, message = '-'  <br> - 오류 발생인 경우, message = 오류 내용과 stack trace 내용 포함 <br> - 추후 생성 예정|
|[**UpdateEps**](Project.md#UpdateEps) | **POST** mrms/eps_update |모델 학습시 계산된 eps 저장| 
|[**UpdateLearnResult**](Project.md#UpdateLearnResult) | **POST** /mrms/learn_result_update |최종결과 DB 저장|
|[**UpdateTime**](Project.md#UpdateTime) | **POST** /mrms/time_update |- 모델 학습 시간 업데이트 <br> - type은 2가지로 분류 <br>&nbsp; "start" : 시작시간 <br>&nbsp; "end" : 종료시간 |
|[**GetProjectInfo**](Project.md#GetProjectInfo) | **GET** /mrms/get_projects_info |모든 프로젝트 정보 반환|
|[**UpdateProjectSttus**](Project.md#UpdateProjectSttus) | **POST** /mrms/update_projects_status |프로젝트 상태 업데이트| 
|[**LearnCreate**](Project.md#LearnCreate) | **POST** /mrms/insert_learn_hist |HPRS에서 진행할 학습 히스토리 추가|
|[**GetLearningLogs**](Project.md#GetLearningLogs) | **GET** /mrms/get_learn_logs |Learn-Job의 MLPS 로그 반환 : 분산으로 동작시 worker 별로| 
|[**GetUUID**](Project.md#GetUUID) | **GET** /mrms/get_uuid |UUID 반환|
|[**ModelResource**](Project.md#ModelResource) | **POST**  /mrms/model_resources |MLPS와의 자원 사용량 통신|
|[**GetHistNoStatus**](Project.md#GetHistNoStatus) | **GET** /mrms/get_histno_status |{"learn_hist_no": (String), <br> "learn_sttus_cd": (String)}|
|[**GetModelsInfo**](Project.md#GetModelsInfo) | **GET** /mrms/get_models_info |프로젝트에 포함된 모든 모델의 정보 반환 <br> [{"learn_hist_no":(String) ,"start_time":"20211014170532", <br> "end_time":"20211015170532", "learn_sttus_cd":"6", "message":"-", ... }]|
|[**GetProjectID**](Project.md#GetProjectID) | **GET** /mrms/get_project_id |프로젝트 id 반환|
|[**UpdateModelAlias**](Project.md#UpdateModelAlias) | **POST** /mrms/model_alias_update |model_alias 업데이트|
|[**UpdateBookmark**](Project.md#UpdateBookmark) | **GET** /mrms/bookmark_update |model_bookmark 업데이트|
|[**ModelDownload**](Project.md#ModelDownload) | **GET** /mrms/model_download |학습모델 export 요청|
|[**UpdateSelectYN**](Project.md#UpdateSelectYN) | **GET** /mrms/update_model_select |모델 선택여부(select_yn) 업데이트|


<a name="ProjectCreate"></a>
# **ProjectCreate**

### Parameters
```
Body: {"project_id" : (String),
       "project_purpose_cd" : (String),
       "dataset_id" : (String),
       "status" : (String),
       "start_time" : (String, YYYYmmddHHMMSS),
       "modeling_mode" : (String),
       "project_target_field" : (String),
       "early_stop_param" : (JSON Format String),
       "tag" : (String)}
```

### Return Type
```
success : "1"
error : "error"
```

<a name="GetProjectSttusCd"></a>
# **GetProjectSttusCd**

### Parameters
```
?project_id=(String)
```

### Return Type
```
success : Status (String)
error : "error"
```

<a name="GetModelSttusCd"></a>
# **GetModelSttusCd**

### Parameters
```
Body: {"hist_no": (String)}
```

### Return Type
```
success : Status (String)
error : "error"
```

<a name="UpdateModelSttus"></a>
# **UpdateModelSttus**

### Parameters
```
Body: {"hist_no": (String),
       "task_idx": (String),
       "learn_sttus_cd: (String),
       "message": (String)}
```

### Return Type
```
success : "1"
error : "error"
```

<a name="UpdateEps"></a>
# **UpdateEps**

### Parameters
```
Body: {"eps" : (String),
       "hist_no" : (String)}
```

### Return Type
```
success : "1"
error : "error"
```

<a name="UpdateLearnResult"></a>
# **UpdateLearnResult**

### Parameters
```
Body: {"result": (JSON Format String),
       "hist_no": (String)}
```

### Return Type
```
success : "1"
error : "error"
```

<a name="UpdateTime"></a>
# **UpdateTime**

### Parameters
```
Body: {"type": (String),
       "curr_time": (String, YYYYmmddHHMMSS),
       "hist_no" : (String)}
```

### Return Type
```
success : "1"
error : "error"
```

<a name="GetProjectInfo"></a>
# **GetProjectInfo**

### Parameters
```
-
```

### Return Type
```
success : List<Json> Format (String)
          ex) [{"project_id":"3290580239","project_purpose":"1",
                "status":"1", "start_time":"20211014170532",
                "end_time":"20211014170532"}]
error : "error"
```

<a name="UpdateProjectSttus"></a>
# **UpdateProjectSttus**

### Parameters
```
Body: {"project_id": (String)
       "status": (String)}
```

### Return Type
```
success : "1"
error : "error"
```

<a name="LearnCreate"></a>
# **LearnCreate**

### Parameters
```
Body: {"param_id": (String),
       "learn_alg_id": (String),
       "data_process_id": (String),
       "project_id": (String)}
```

### Return Type
```
success : "1"
error : "error"
```

<a name="GetLearningLogs"></a>
# **GetLearningLogs**

### Parameters
```
?id=(String)&tail=(true or false)
```

### Return Type
```
success : Json Format(String)
          ex) {"worker_idx": "log string"}
error : "error"
```

<a name="GetUUID"></a>
# **GetUUID**

### Parameters
```
-
```

### Return Type
```
success : uuid
error : null
```

<a name="ModelResource"></a>
# **ModelResource**

### Parameters
```
Body: {"learn_hist_no": (String),
       "cpu": {"percent": (String)}}
```

### Return Type
```
success : "1"
error : "error"
```

<a name="GetHistNoStatus"></a>
# **GetHistNoStatus**

### Parameters
```
?project_id=(String)
```

### Return Type
```
success : Json Format(String)
error : "error"
```

<a name="GetModelsInfo"></a>
# **GetModelsInfo**

### Parameters
```	
?project_id=(String)&learn_hist_no=(String) 또는
?project_id=(String)
```

### Return Type
```
success : List<Json> Format (String)
error : "error"
```

<a name="GetProjectID"></a>
# **GetProjectID**

### Parameters
```
?hist_no=(String)
```

### Return Type
```
success : project id(String)
error : "error"
```

<a name="UpdateModelAlias"></a>
# **UpdateModelAlias**

### Parameters
```
Body: {"learn_hist_no": (String),
       "model_alias": (String)}
```

### Return Type
```
success : "1"
error : "error"
```

<a name="UpdateBookmark"></a>
# **UpdateBookmark**

### Parameters
```
?bookmarkYN=(String)&learn_hist_no=(String)
```

### Return Type
```
success : "1"
error : "error"
```

<a name="ModelDownload"></a>
# **ModelDownload**

### Parameters
```
?learn_hist_no=(String)
```

### Return Type
```
success : file download
error : exception
```

<a name="UpdateSelectYN"></a>
# **UpdateSelectYN**

### Parameters
```
?learn_hist_no=(String)&project_id=(String)
```

### Return Type
```
success : "1"
error : "error"
```

