# REST API - DataAnalysis

> Kubernetes Pod 내부 : *http://mrms-svc:9200* </br>
> Kubernetes Pod 외부 : *http://marster-ip:31920*


|Method|HTTP request|Description|
|------|------------|-----------|
[**GetDataAnalsInfo**](DataAnalysis.md#GetDataAnalsInfo) | **POST** /mrms/get_data_anls_info |데이터셋 별 데이터 분석 상세 정보 반환|
[**InsertDataAnalsInfo**](DataAnalysis.md#InsertDataAnalsInfo) | **POST** /mrms/insert_data_anls_info |데이터 분석 결과 업데이트|
[**InsertDataAnalsInfo**](DataAnalysis.md#InsertDataAnalsInfo) | **GET** /mrms/insert_data_anls_info |데이터 분석 결과 반환|
[**DAWorkerReq**](DataAnalysis.md#DAWorkerReq) | **GET** /request_da_worker |DA Worker 생성요청|
[**GetFieldUnique**](DataAnalysis.md#GetFieldUnique) | **GET** 	/mrms/get_field_unique |unique 반환|
[**GetFieldWord**](DataAnalysis.md#GetFieldWord) | **GET** /mrms/get_field_word |word 반환|
[**GetDatasetMeta**](DataAnalysis.md#GetDatasetMeta) | **GET** /mrms/get_dataset_meta |데이터셋 EDA결과 정보 반환 |


<a name="GetDataAnalsInfo"></a>
# **GetDataAnalsInfo**
데이터셋 ID를 통해 데이터 분석 상세 정보 반환
### Parameters
```
Body: {"dataset_id": (String)}
```
- **dataset_id**
```
데이터셋 ID
```
### Return Type
```
success : Json Format (String)
          {"data_analysis_id": (String),
           "metadata_json": (JSON format String),
           "label_yn": (String) ,
           "dist_file_cnt": (String),
           ...}
error : "error"
```

<a name="InsertDataAnalsInfo"></a>
# **InsertDataAnalsInfo**
데이터 분석 후 결과 업데이트
### Parameters
```
Body: {"data_analysis_id": (String),
       "metadata_json": (JSON format String),
       "dist_file_cnt": (String),
       "analysis_file_nm": (String),
       "dataset_id": (String),
       "dataset_meta_json": (JSON format String)}
```
- **data_analysis_id**
```
데이터 분석 ID
```
- **metadata_json**
```
메타데이터 JSON(데이터 분석 결과 정보)
ex) {
      "dataset_meta": {
        "features": 6,
        "instances": 891
      },
      "file_num_line": [
        891
      ],
      "file_list": [
        (String)
      ],
      "meta": [
        {
          "field_nm": "Survived",
          "field_idx": 0,
          "field_type": "int",
          "type_stat": {
            "null": 0,
            "int": 891,
            "float": 0,
            "string": 0,
            "date": 0,
            "list": 0
          },
          "statistics": {
            "average": 0.3838383838383838,
            "max": 1,
            "min": 0,
            "sum": 342,
            "unique": {
              "unique": {
                "0": 549,
                "1": 342
              },
              "unique_count": 2
            },
            "is_category": "True",
            "valueratio": {
              "0": 0.62,
              "1": 0.38
            },
            "variance": 0.2365064789307223,
            "std_dev": 0.48631931786710086,
            "kurtosis": -1.7717860224331479,
            "skewness": 0.47771746625684985
          }
        },
     ... 
    }
```
- **dist_file_cnt**
```
분산 파일 개수 
```
- **analysis_file_nm**
```
데이터 분석 파일명
```
- **dataset_id**
```
데이터셋 ID
```
- **dataset_meta_json**
```
데이터셋 메타데이터 JSON(EDA 결과 정보)
ex) [
      {
        "mutual_information": [
          0.9607079018756469,
          null,
          null,
          null,
          null,
          null
        ],
        "covariance": [
          0.2365064789307223,
          -0.1375483227335088,
          null,
          -0.018932308494598187,
          0.03198086363069515,
          6.214803899828815
        ],
        "correlation": [
          1,
          -0.3384810359610158,
          null,
          -0.03532249888573589,
          0.08162940708348222,
          0.2573065223849618
        ]
      },
     ... 
    ]
```
### Return Type
```
success : "1"
error : "error"
```

<a name="InsertDataAnalsInfo"></a>
# **InsertDataAnalsInfo**
데이터셋 ID를 통해 데이터 분석 결과 반환
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
success : "1"
error : "error"
```

<a name="DAWorkerReq"></a>
# **DAWorkerReq**
데이터셋 ID와 워커 수에 따른 DA Worker 생성 요청
### Parameters
```
?id=(String)&num_worker=(String)
```
- **id**
```
데이터셋 ID(dataset_id)
```
- **num_worker**
```
워커 수 
```
### Return Type
```
success : "1"
error : "error"
```

<a name="GetFieldUnique"></a>
# **GetFieldUnique**
데이터셋 ID와 필드명을 통해 unique 정보 반환(고유한 행의 갯수, 고유한 종류 갯수)
### Parameters
```
?dataset_id=(String)&field_name=(String)
```
- **id**
```
데이터셋 ID(dataset_id)
```
- **field_name**
```
필드명(field_nm)
```
### Return Type
```
success : Json Format (String)
          {"unique": {
             "0": 549,
             "1": 342
           },
             "unique_count": 2
          }
error : "error"
```

<a name="GetFieldWord"></a>
# **GetFieldWord**
데이터셋 ID와 필드명을 통해 word 정보 반환(출현단어 갯수)
### Parameters
```
?dataset_id=(String)&field_name=(String)
```
- **id**
```
데이터셋 ID(dataset_id)
```
- **field_name**
```
필드명(field_nm)
```
### Return Type
```
success : Json Format (String)
          {"viewts": 23068,
           "lq2cr": 2779,
           "pfno": 17481,
           "opg": 3150,
           "c19": 38157,
           "bqp5f": 207,
           "as": 28127,
           ...
          }
error : "error"
```

<a name="GetDatasetMeta"></a>
# **GetDatasetMeta**
데이터셋 ID를 통해 데이터셋 EDA 결과 정보 반환
### Parameters
```
?dataset_id=(String)
```
- **id**
```
데이터셋 ID(dataset_id)
```
### Return Type
```
success : Json Format (String)
        [
          {
            "mutual_information": [
              0.9607079018756469,
              null,
              null,
              null,
              null,
              null
            ],
            "covariance": [
              0.2365064789307223,
              -0.1375483227335088,
              null,
              -0.018932308494598187,
              0.03198086363069515,
              6.214803899828815
            ],
            "correlation": [
              1,
              -0.3384810359610158,
              null,
              -0.03532249888573589,
              0.08162940708348222,
              0.2573065223849618
            ]
          },
         ... 
        ]
error : "error"
```
