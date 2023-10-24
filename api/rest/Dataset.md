# REST API - Dataset

> Kubernetes Pod 내부 : *http://mrms-svc:9200* </br>
> Kubernetes Pod 외부 : *http://marster-ip:31920*


|Method|HTTP request|Description|
|------|------------|-----------|
|[**InsertDataset**](Dataset.md#InsertDataset) | **POST** /mrms/insert_dataset |데이터셋 추가|
|[**DeleteDataset**](Dataset.md#DeleteDataset) | **POST** /mrms/delete_dataset |데이터셋 삭제|
|[**GetDatasetInfo**](Dataset.md#GetDatasetInfo) | **POST** /mrms/get_dataset_info |데이터셋 별 상태값 반환|
|[**UpdateDatasetSttus**](Dataset.md#UpdateDatasetSttus) | **POST** /mrms/update_dataset_status |데이터셋 상태 업데이트|
|[**GetDatasetFormat**](Dataset.md#GetDatasetFormat) | **GET** /mrms/get_dataset_format | 데이터셋 포맷 변환|
|[**GetTargetField**](Dataset.md#GetTargetField) | **POST** /mrms/get_target_field | target field 반환|


<a name="InsertDataset"></a>
# **InsertDataset**

### Parameters
```
Body: {"dataset_id": (String),
       "dataset_format": (String),
       "n_cols": (String) ,
       "dataset_size": (String),
       "n_rows": (String),
       "status_cd": (String),
       "format_json": (JSON format String)}
```

### Return Type
```
success : "1"
error : "error"
```

<a name="DeleteDataset"></a>
# **DeleteDataset**

### Parameters
```
Body : {"dataset_id": (String)}
```

### Return Type
```
success : "1"
error : "error"
```

<a name="GetDatasetInfo"></a>
# **GetDatasetInfo**

### Parameters
```
Body : {"dataset_id": (String)}
```

### Return Type
```
success : Json Format (String)
          ex) {"dataset_id":"135315", "status":"3",...}
error : "error"
```

<a name="UpdateDatasetSttus"></a>
# **UpdateDatasetSttus**

### Parameters
```
Body : {"dataset_id": (String),
        "status_cd": (String) }
```

### Return Type
```
success : "1"
error : "error"
```

<a name="GetDatasetFormat"></a>
# **GetDatasetFormat**

### Parameters
```
?data_analysis_id=(String)
```

### Return Type
```
success : "1"
error : "error"
```

<a name="GetTargetField"></a>
# **GetTargetField**

### Parameters
```
Body : {"project_id": (String)}
```

### Return Type
```
success : project_target_fileld(String)
error : "error"
```
