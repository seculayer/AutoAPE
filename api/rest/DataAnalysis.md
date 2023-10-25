# REST API - DataAnalysis

> Kubernetes Pod 내부 : *http://mrms-svc:9200* </br>
> Kubernetes Pod 외부 : *http://marster-ip:31920*


|Method|HTTP request|Description|
|------|------------|-----------|
[**GetDataAnalsInfo**](DataAnalysis.md#GetDataAnalsInfo) | **POST** /mrms/get_data_anls_info |데이터셋 별 데이터 분석정보 반환|
[**InsertDataAnalsInfo**](DataAnalysis.md#InsertDataAnalsInfo) | **POST** /mrms/insert_data_anls_info |데이터 분석 후 결과 저장|
[**InsertDataAnalsInfo**](DataAnalysis.md#InsertDataAnalsInfo) | **GET** /mrms/insert_data_anls_info |데이터 분석 후 결과 저장|
[**DAWorkerReq**](DataAnalysis.md#DAWorkerReq) | **GET** /request_da_worker |DA Worker 생성요청|
[**GetFieldUnique**](DataAnalysis.md#GetFieldUnique) | **GET** 	/mrms/get_field_unique |unique 반환|
[**GetFieldWord**](DataAnalysis.md#GetFieldWord) | **GET** /mrms/get_field_word |word 반환|


<a name="GetDataAnalsInfo"></a>
# **GetDataAnalsInfo**

### Parameters
```
Body: {"dataset_id": (String)}
```

### Return Type
```
success : Json Format (String)
          ex) {"data_analysis_id":"135315",...}
error : "error"
```

<a name="InsertDataAnalsInfo"></a>
# **InsertDataAnalsInfo**

### Parameters
```
Body: {"data_analysis_id": (String),
       "metadata_json": (String),
       "label_yn": (String),
       "dist_file_cnt": (String),
       "analysis_file_nm": (String),
       "dataset_id": (String)}
```

### Return Type
```
success : "1"
error : "error"
```

<a name="InsertDataAnalsInfo"></a>
# **InsertDataAnalsInfo**

### Parameters
```
?dataset_id=(String)
```

### Return Type
```
success : "1"
error : "error"
```

<a name="DAWorkerReq"></a>
# **DAWorkerReq**

### Parameters
```
?id=(String)&num_worker=(String)
```

### Return Type
```
success : "1"
error : "error"
```

<a name="GetFieldUnique"></a>
# **GetFieldUnique**

### Parameters
```
?dataset_id=(String)&field_name=(String)
```

### Return Type
```
success : Json Format(String)
error : "error"
```

<a name="GetFieldWord"></a>
# **GetFieldWord**

### Parameters
```
?dataset_id=(String)&field_name=(String)
```

### Return Type
```
success : Json Format(String)
error : "error"
```