# REST API - Dataset

> Kubernetes Pod 내부 : *http://mrms-svc:9200* </br>
> Kubernetes Pod 외부 : *http://marster-ip:31920*


|Method|HTTP request|Description|
|------|------------|-----------|
|[**InsertDataset**](Dataset.md#InsertDataset) | **POST** /mrms/insert_dataset |데이터셋 추가|
|[**DeleteDataset**](Dataset.md#DeleteDataset) | **POST** /mrms/delete_dataset |데이터셋 삭제|
|[**GetDatasetInfo**](Dataset.md#GetDatasetInfo) | **POST** /mrms/get_dataset_info |데이터셋 상세 정보 반환|
|[**UpdateDatasetSttus**](Dataset.md#UpdateDatasetSttus) | **POST** /mrms/update_dataset_status |데이터셋 상태 업데이트|
|[**GetDatasetFormat**](Dataset.md#GetDatasetFormat) | **GET** /mrms/get_dataset_format | 데이터셋 형태 코드 반환|
|[**GetTargetField**](Dataset.md#GetTargetField) | **POST** /mrms/get_target_field | 타겟 필드 반환|


<a name="InsertDataset"></a>
# **InsertDataset**
데이터셋 형태, 크기, 상태 등에 대한 상세 정보를 포함한 새로운 데이터셋 추가
### Parameters
```
Body: {"dataset_id": (String),
       "dataset_format": (String),
       "n_cols": (String) ,
       "dataset_size": (String),
       "n_rows": (String),
       "status_cd": (String),
       "format_json": (JSON format String),
       "target_field": (String}
```
- **dataset_id**
```
데이터셋 ID
```
- **dataset_format**
```
데이터셋 형태 코드

1(TEXT)
2(IMAGE)
3(TABLE)
```
- **n_cols**
```
컬럼 수
```
- **dataset_size**
```
데이터셋 크기(Byte)
```
- **n_rows**
```
데이터 개수
```
- **status_cd**
```
상태코드

1(업로드 대기)
2(업로드 진행)
3(업로드 완료)
4(업로드에러)
5(데이터 분석 요청)
6(데이터 분석 중)
7(데이터 분석 완료)
8(데이터 분석 에러)
9(데이터 분석 파드 삭제 요청)
```
- **format_json**
```
파일형태 상세 정보 JSON
ex) {"file_path": (String),
     "file_nm": (String),
     "field_list": "Survived,Pclass,Sex,SibSp,Parch,Fare"}
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

<a name="DeleteDataset"></a>
# **DeleteDataset**
데이터셋 ID를 통해 데이터셋 삭제
### Parameters
```
Body : {"dataset_id": (String)}
```
- **dataset_id**
```
데이터셋 ID
```
### Return Type
```
success : "1"
error : "error"
```

<a name="GetDatasetInfo"></a>
# **GetDatasetInfo**
데이터셋 ID를 통해 데이터셋 상세 정보 반환
### Parameters
```
Body : {"dataset_id": (String)}
```
- **dataset_id**
```
데이터셋 ID
```
- **status_cd**
```
상태코드

1(업로드 대기)
2(업로드 진행)
3(업로드 완료)
4(업로드에러)
5(데이터 분석 요청)
6(데이터 분석 중)
7(데이터 분석 완료)
8(데이터 분석 에러)
9(데이터 분석 파드 삭제 요청)
```
### Return Type
```
success : Json Format (String)
      ex) {"dataset_id": (String),
           "dataset_format": (String),
           "n_cols": (String) ,
           "dataset_size": (String),
           "n_rows": (String),
           ...
          }
error : "error"
```

<a name="UpdateDatasetSttus"></a>
# **UpdateDatasetSttus**
현재 데이터셋 상태를 실시간으로 해당 메소드 요청을 통해 업데이트
### Parameters
```
Body : {"dataset_id": (String),
        "status_cd": (String) }
```
- **dataset_id**
```
데이터셋 ID
```
- **status_cd**
```
상태코드

1(업로드 대기)
2(업로드 진행)
3(업로드 완료)
4(업로드에러)
5(데이터 분석 요청)
6(데이터 분석 중)
7(데이터 분석 완료)
8(데이터 분석 에러)
9(데이터 분석 파드 삭제 요청)
```
### Return Type
```
success : "1"
error : "error"
```

<a name="GetDatasetFormat"></a>
# **GetDatasetFormat**
데이터 분석 ID를 통해 데이터셋 형태 코드(dataset_format) 반환
### Parameters
```
?data_analysis_id=(String)
```
- **data_analysis_id**
```
데이터분석 ID
```
### Return Type
```
success : "1"
error : "error"
```

<a name="GetTargetField"></a>
# **GetTargetField**
프로젝트 ID를 통해 타겟 필드(target_field) 반환
### Parameters
```
Body : {"project_id": (String)}
```
- **project_id**
```
프로젝트 ID
```
### Return Type
```
success : target_field(String)
error : "error"
```

