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
|[**GetXAIInfo**](XAI.md#GetXAIInfo) | **GET** /mrms/get_xai_info |XAI 정보 반환|


<a name="XAICreate"></a>
# **XAICreate**
XAI 생성
### Parameters
```
?infr_hist_no=(String)&target_field=(String)&data_analysis_id=(String)&learn_hist_no=(String)
```
- **infr_hist_no**
``` 
검증 Hist 번호(infr_hist_no)
```
- **target_field**
```
지정된 타겟 필드
```
- **data_analysis_id**
```
데이터 분석 ID
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

<a name="UpdateXAICreateYN"></a>
# **UpdateXAICreateYN**
검증 HIST 번호에 따른 XAI 생성 여부(xai_create_yn) 업데이트
### Parameters
```
?infr_hist_no=(String)
```
- **infr_hist_no**
``` 
검증 Hist 번호(infr_hist_no)
```
### Return Type
```
success : "1"
error : "error"
```

<a name="XAIProgress"></a>
# **XAIProgress**
XAI Hist 번호를 통해 XAI 진행률 반환
### Parameters
```
?id=(String)
```
- **id**
```
XAI Hist 번호(xai_hist_no)
```
### Return Type
```
success : xai progress rate(String)
error : "error"
```

<a name="XAIProgress"></a>
# **XAIProgress**
XAI가 시작될 때, "update" 모드로 전달받은 값을 통해 실시간으로 XAI 진행률 정보를 업데이트
<br> XAI가 완료된 경우, "delete" 모드로 전달받은 값을 사용하여 해당 XAI HIST 번호에 대한 XAI 진행률 정보를 제거
### Parameters
```
Body: {"xai_hist_no": (String),
       "progress_rate": (String),
       "mode": (String)}
```
- **xai_hist_no**
```
XAI Hist 번호
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

<a name="UpdateXAISttus"></a>
# **UpdateXAISttus**
현재 XAI 상태를 실시간으로 해당 메소드 요청을 통해 업데이트
### Parameters
```
Body: {"hist_no": (String),
       "sttus_cd": (String)}
```
- **hist_no**
```
XAI Hist 번호(xai_hist_no)
```
- **sttus_cd**
```
XAI상태코드(xai_sttus_cd)

1: XAI 요청
2: XAI 대기
5: XAI 진행
6: XAI 완료
7: XAI 에러
```
### Return Type
```
success : "1"
error : "error"
```

<a name="GetXAIInfo"></a>
# **GetXAIInfo**
프로젝트 ID를 통해 XAI에 대한 정보 반환
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
      ex) [{"xai_hist_no": (String),
           "xai_sttus_cd": (String),
           "alg_nm": (String),
           "alg_cont": (String),
           "learn_hist_no": (String)}
          ]
error : "error"
```

