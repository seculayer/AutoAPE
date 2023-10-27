# REST API - Project

> Kubernetes Pod 내부 : *http://mrms-svc:9200* </br>
> Kubernetes Pod 외부 : *http://marster-ip:31920*


|Method|HTTP request|Description|
|------|------------|-----------|
|[**ProjectCreate**](Project.md#ProjectCreate) | **POST** /mrms/project_create |- 신규 Project DB 추가 요청 <br> - modeling_mode <br>&nbsp; 1: 기본, 2: 재학습 <br> - early_stop_param <br>&nbsp; {"early_type" : 0,1,2,3 (string), # 0: None, 1: min, 2: max, 3: variance <br>&nbsp; "early_key" : (string), # "accuracy" or "loss" or "val_loss" or "val_accuracy" <br>&nbsp; "early_value" : (float), # 0에서 1 사이의 수 <br>&nbsp; "minsteps" : (int) # 최소 스텝 수} <br> - tag <br>&nbsp; comma(",") split string


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