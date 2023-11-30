# REST API - Project

> Kubernetes Pod 내부 : *http://mrms-svc:9200* </br>
> Kubernetes Pod 외부 : *http://marster-ip:31920*


|Method|HTTP request|Description|
|------|------------|-----------|
|[**ProjectCreate**](Project.md#ProjectCreate) | **POST** /mrms/project_create |신규 Project DB 추가 요청|
|[**GetProjectSttusCd**](Project.md#GetProjectSttusCd) | **GET**  /mrms/get_proj_sttus_cd |해당 프로젝트 현재 상태 반환| 
|[**GetModelSttusCd**](Project.md#GetModelSttusCd) | **POST** /mrms/get_sttus_cd |해당 학습 HIST 번호의 학습 상태 반환|
|[**UpdateModelSttus**](Project.md#UpdateModelSttus) | **POST** /mrms/status_update/learn |학습 상태 업데이트|
|[**UpdateEps**](Project.md#UpdateEps) | **POST** mrms/eps_update |모델 학습시 계산된 eps 업데이트| 
|[**UpdateLearnResult**](Project.md#UpdateLearnResult) | **POST** /mrms/learn_result_update |최종 결과 업데이트|
|[**UpdateTime**](Project.md#UpdateTime) | **POST** /mrms/time_update |모델 학습 시간 업데이트|
|[**GetProjectInfo**](Project.md#GetProjectInfo) | **GET** /mrms/get_projects_info |프로젝트 정보 반환|
|[**UpdateProjectSttus**](Project.md#UpdateProjectSttus) | **POST** /mrms/update_projects_status |프로젝트 상태 업데이트| 
|[**LearnCreate**](Project.md#LearnCreate) | **POST** /mrms/insert_learn_hist |HPRS에서 진행할 학습 히스토리 추가|
|[**GetLearningLogs**](Project.md#GetLearningLogs) | **GET** /mrms/get_learn_logs |Learn-Job의 MLPS 로그 반환| 
|[**GetUUID**](Project.md#GetUUID) | **GET** /mrms/get_uuid |UUID 반환|
|[**ModelResource**](Project.md#ModelResource) | **POST**  /mrms/model_resources |MLPS와의 자원 사용량 통신|
|[**GetHistNoStatus**](Project.md#GetHistNoStatus) | **GET** /mrms/get_histno_status |학습 상태 반환(프로젝트 ID로 검색)|
|[**GetModelsInfo**](Project.md#GetModelsInfo) | **GET** /mrms/get_models_info |학습한 모델 상세 정보 반환|
|[**GetProjectID**](Project.md#GetProjectID) | **GET** /mrms/get_project_id |프로젝트 ID 반환|
|[**UpdateModelAlias**](Project.md#UpdateModelAlias) | **POST** /mrms/model_alias_update |model alias 업데이트|
|[**UpdateBookmark**](Project.md#UpdateBookmark) | **GET** /mrms/bookmark_update |model bookmark 업데이트|
|[**ModelDownload**](Project.md#ModelDownload) | **GET** /mrms/model_download |학습한 모델 export 요청|
|[**UpdateSelectYN**](Project.md#UpdateSelectYN) | **GET** /mrms/update_model_select |모델 선택여부(select_yn) 업데이트|


<a name="ProjectCreate"></a>
# **ProjectCreate**
프로젝트 생성
### Parameters
```
Body: {"project_id" : (String),
       "project_purpose_cd" : (String),
       "dataset_id" : (String),
       "status" : (String),
       "start_time" : (String, YYYYmmddHHMMSS),
       "modeling_mode" : (String),
       "early_stop_param" : (JSON Format String),
       "tag" : (String)}
```
- **project_id**
```
프로젝트 ID
```
- **project_purpose_cd**
```
프로젝트 목적코드

1: classification
10: trend analysis
```
- **dataset_id**
```
데이터셋 ID
```
- **status**
```
상태코드

1: 업로드 대기
2: 업로드 진행
3: 업로드 완료
4: 업로드 에러
5: 데이터 분석 요청
6: 데이터 분석 중
7: 데이터 분석 완료
8: 데이터 분석 에러
9: 데이터 분석 파드 삭제 요청
```
- **start_time**
```
시작시간
```
- **modeling_mode**
```
1: 기본 
2: 재학습
```
- **early_stop_param**
```
{"early_type" : (string),  # 0: None, 1: min, 2: max, 3: variance
 "early_key" : (string),   # "accuracy" or "loss" or "val_loss" or "val_accuracy"
 "early_value" : (float),  # 0에서 1 사이의 수
 "minsteps" : (int)}       # 최소 스텝 수
}
```
- **tag**
```
comma(",") split string
```
### Return Type
```
success : "1"
error : "error"
```

<a name="GetProjectSttusCd"></a>
# **GetProjectSttusCd**
프로젝트 ID를 통해 상태 코드(status)를 반환
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
success : Status (String)
error : "error"
```

<a name="GetModelSttusCd"></a>
# **GetModelSttusCd**
학습 HIST 번호를 통해 학습 상태(learn_sttus_cd)를 반환
<br> project status와는 별개
### Parameters
```
Body: {"hist_no": (String)}
```
- **learn_hist_no**
```
학습 Hist 번호
```
### Return Type
```
success : Status (String)
error : "error"
```

<a name="UpdateModelSttus"></a>
# **UpdateModelSttus**
현재 학습 상태를 실시간으로 해당 메소드 요청을 통해 업데이트
### Parameters
```
Body: {"hist_no": (String),
       "task_idx": (String),
       "sttus_cd: (String),
       "message": (String)}
```
- **hist_no**
```
학습 Hist 번호(learn_hist_no)
```
- **task_idx**
```
이슈워커인덱스(issue_task_idx)
```
- **sttus_cd**
```
학습 상태코드(learn_sttus_cd)

5: 학습 진행
6: 학습 완료
7: 학습 에러
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

<a name="UpdateEps"></a>
# **UpdateEps**
모델 학습시 계산된 eps를 실시간으로 해당 메소드 요청을 통해 업데이트
### Parameters
```
Body: {"eps" : (String),
       "hist_no" : (String)}
```
- **eps**
```
데이터 처리 속도
```
- **hist_no**
```
학습 Hist 번호(learn_hist_no)
```
### Return Type
```
success : "1"
error : "error"
```

<a name="UpdateLearnResult"></a>
# **UpdateLearnResult**
최종 학습 결과를 실시간으로 해당 메소드 요청을 통해 업데이트
### Parameters
```
Body: {"result": (JSON Format String),
       "hist_no": (String)}
```
- **result**
```
학습 결과 JSON(ml_result_json)
ex) {"loss": 0.406673789024353,
     "accuracy": 0.8798828125,
     "val_loss": 0.32281994819641113,
     "val_accuracy": 0.91015625,
     "step": 1
    }
```
- **hist_no**
```
학습 Hist 번호(learn_hist_no)
```
### Return Type
```
success : "1"
error : "error"
```

<a name="UpdateTime"></a>
# **UpdateTime**
모델 학습 시간을 실시간으로 해당 메소드 요청을 통해 업데이트
<br> type에 따라서 검증 시작(updateStartInfrTime)과 검증 종료(updateEndInfrTime) 각각 진행
### Parameters
```
Body: {"type": (String),
       "curr_time": (String, YYYYmmddHHMMSS),
       "hist_no" : (String)}
```
- **type**
```
"start": 시작시간
"end": 종료시간
```
- **curr_time**
```
updateStartTime: 시작시간(start_time)
updateEndInfrTime: 종료시간(end_time)
```
- **hist_no**
```
학습 Hist 번호(learn_hist_no)
```
### Return Type
```
success : "1"
error : "error"
```

<a name="GetProjectInfo"></a>
# **GetProjectInfo**
프로젝트 상세 정보 반환
### Parameters
```
-
```
### Return Type
```
success : List<Json> Format (String)
      ex) [{"project_id": (String),
            "project_purpose_cd": (String),
            "data_analysis_id": (String),
            "status": (String), 
            "start_time": (String),
            "end_time": (String)}
          ]
error : "error"
```

<a name="UpdateProjectSttus"></a>
# **UpdateProjectSttus**
프로젝트 상태를 실시간으로 해당 메소드 요청을 통해 업데이트
### Parameters
```
Body: {"project_id": (String)
       "status": (String)}
```
- **project_id**
```
프로젝트 ID
```
- **status**
```
상태코드

1: 업로드 대기
2: 업로드 진행
3: 업로드 완료
4: 업로드 에러
5: 데이터 분석 요청
6: 데이터 분석 중
7: 데이터 분석 완료
8: 데이터 분석 에러
9: 데이터 분석 파드 삭제 요청
```
### Return Type
```
success : "1"
error : "error"
```

<a name="LearnCreate"></a>
# **LearnCreate**
HPRS(하이퍼 파라미터 추천)에서 진행할 학습 히스토리 추가
### Parameters
```
Body: {"learn_hist_no": (String),
       "param_id": (String),
       "alg_anal_id": (String),
       "alg_id": (String),
       "dp_analysis_id": (String),
       "project_id": (String),
       "learn_sttus_cd": (String),
       "start_time": (String)}
```
- **learn_hist_no**
```
학습 HIST 번호
```
- **param_id**
```
파라미터 ID
```
- **alg_anal_id**
```
알고리즘 분석 ID
```
- **alg_id**
```
알고리즘 ID
```
- **dp_analysis_id**
```
데이터 처리 분석 ID
```
- **project_id**
```
프로젝트 ID
```
- **learn_sttus_cd**
```
학습 상태코드

5: 학습 진행
6: 학습 완료
7: 학습 에러
```
- **start_time**
```
시작 시간
```
### Return Type
```
success : "1"
error : "error"
```

<a name="GetLearningLogs"></a>
# **GetLearningLogs**
Learn-Job의 MLPS 로그 반환 : 분산으로 동작시 worker 별로
### Parameters
```
?id=(String)&tail=(true or false)
```
- **id**
```
학습 HIST 번호(learn_hist_no)
```
### Return Type
```
success : Json Format (String)
      ex) {"worker_idx": "log string"}
error : "error"
```

<a name="GetUUID"></a>
# **GetUUID**
UUID 반환
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
MLPS와의 자원 사용량(cpu) 통신
### Parameters
```
Body: {"learn_hist_no": (String),
       "cpu": {"percent": (String)}}
```
- **learn_hist_no**
```
학습 HIST 번호
```
### Return Type
```
success : "1"
error : "error"
```

<a name="GetHistNoStatus"></a>
# **GetHistNoStatus**
프로젝트 ID를 통해 학습 HIST 번호와 학습 상태 코드를 반환
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
success : List<Json> Format (String)
      ex) [{"learn_hist_no": (String),
           "learn_sttus_cd": (String)}
          ]
error : "error"
```

<a name="GetModelsInfo"></a>
# **GetModelsInfo**
프로젝트 ID를 통해 학습한 모델 상세 정보 반환
### Parameters
```	
?project_id=(String)&learn_hist_no=(String) 또는
?project_id=(String)
```
- **project_id**
```
프로젝트 ID
```
- **learn_hist_no**
```
학습 HIST 번호
```
### Return Type
```
success : List<Json> Format (String)
      ex) [
            {"learn_hist_no":(String),
             "start_time":"20211014170532",
             "end_time":"20211015170532", 
             "learn_sttus_cd":"6", 
             "message":"-", 
             ... }
           ]
error : "error"
```

<a name="GetProjectID"></a>
# **GetProjectID**
학습 HIST 번호를 통해 프로젝트 ID 반환
### Parameters
```
?hist_no=(String)
```
- **hist_no**
```
학습 HIST 번호(learn_hist_no)
```
### Return Type
```
success : project id(String)
error : "error"
```

<a name="UpdateModelAlias"></a>
# **UpdateModelAlias**
model alias 업데이트
### Parameters
```
Body: {"learn_hist_no": (String),
       "model_alias": (String)}
```
- **learn_hist_no**
```
학습 HIST 번호
```
### Return Type
```
success : "1"
error : "error"
```

<a name="UpdateBookmark"></a>
# **UpdateBookmark**
model bookmark 업데이트
### Parameters
```
?bookmarkYN=(String)&learn_hist_no=(String)
```
- **learn_hist_no**
```
학습 HIST 번호
```
### Return Type
```
success : "1"
error : "error"
```

<a name="ModelDownload"></a>
# **ModelDownload**
학습 HIST 번호를 통해 학습한 모델 export 요청
### Parameters
```
?learn_hist_no=(String)
```
- **learn_hist_no**
```
학습 HIST 번호
```
### Return Type
```
success : file download
error : exception
```

<a name="UpdateSelectYN"></a>
# **UpdateSelectYN**
모델 선택여부(select_yn) 업데이트
### Parameters
```
?learn_hist_no=(String)&project_id=(String)
```
- **learn_hist_no**
```
학습 HIST 번호
```
- **project_id**
```
프로젝트 HIST 번호
```
### Return Type
```
success : "1"
error : "error"
```
