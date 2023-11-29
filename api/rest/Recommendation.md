# REST API - Recommendation

> Kubernetes Pod 내부 : *http://mrms-svc:9200* </br>
> Kubernetes Pod 외부 : *http://marster-ip:31920*


|Method|HTTP request|Description|
|------|------------|-----------|
|[**GetVarFuncList**](Recommendation.md#GetVarFuncList) | **GET** /mrms/get_cvt_fn |데이터변환 함수 정보 반환|
|[**InsertDPAnalsInfo**](Recommendation.md#InsertDPAnalsInfo) | **POST** /mrms/insert_dp_anls_info |dprs 결과 추가|
|[**InsertAlgAnlsInfo**](Recommendation.md#InsertAlgAnlsInfo) | **POST** /mrms/insert_alg_anls_info |mars 결과 추가|
|[**GetAlgorithmInfo**](Recommendation.md#GetAlgorithmInfo) | **GET** /mrms/get_algorithm_info |algorithm 정보 반환|
|[**GetParamInfo**](Recommendation.md#GetParamInfo) | **GET** /mrms/get_param_info |특정 알고리즘에 대한 hyper parameter 구성 정보 반환|
|[**InsertMLParamInfo**](Recommendation.md#InsertMLParamInfo) | **POST** /mrms/insert_ml_param_info |프로젝트에서 사용될 파라미터 선택 후 추가|


<a name="GetVarFuncList"></a>
# **GetVarFuncList**
데이터변환 함수 정보 반환(변환함수 클래스명, 태그)
### Parameters
```
-
```
### Return Type
```
success : List<Json> Format (String)
         ex) [{"conv_func_cls":"NotNormal","conv_func_tag":"[[@not_normal()]]"},
             ...
             ]
error : "error"
```

<a name="InsertDPAnalsInfo"></a>
# **InsertDPAnalsInfo**
데이터 전처리 함수 추천 후 전처리 함수 종류, 분석, 정보 등에 대한 상세 정보 추가
### Parameters
```
Body: [{"dp_analysis_id": (String),
       "project_id": (String),
       "data_analysis_json": (JSON format String),
       "data_analysis_id": (String)}
      ]
```
- **dp_analysis_id**
```
데이터 분석 처리 ID
```
- **project_id**
```
프로젝트 ID
```
- **data_analysis_json**
```
데이터 분석 처리 JSON(데이터 전처리 함수 정보 포함)
ex) [
      {
        "name": "Survived",
        "field_sn": 0,
        "field_type": "int",
        "functions": "[[@one_hot_encode()]]",
        "statistic": {
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
    ]
```
- **data_analysis_id**
```
데이터 분석 ID
```
### Return Type
```
success : "1"
error : "error"
```

<a name="InsertAlgAnlsInfo"></a>
# **InsertAlgAnlsInfo**
알고리즘 추천 후 알고리즘 종류, 코드 등에 대한 상세 정보 추가
### Parameters
```
Body: [{"alg_anal_id": (String),
        "project_id": (String),
        "alg_id": (String),
        "dp_analysis_id": (String),
        "data_analysis_json": (JSON format String),
        "alg_type": (String),
        "alg_json": (JSON format String),
        "algorithm_code": (String)}
      ]
```
- **alg_anal_id**
```
알고리즘 분석 ID
```
- **project_id**
```
프로젝트 ID
```
- **alg_id**
```
알고리즘 ID
```
- **dp_analysis_id**
```
데이터 분석 처리 ID
```
- **data_analysis_json**
```
데이터 분석 처리 JSON(데이터 전처리 함수 정보 포함)
```
- **alg_type**
```
알고리즘 종류

1(Classifier)
2(Regressor)
3(Clustering)
4(Word Embedding)
5(Feature Extraction)
7(Outlier Detection)
10(Trend Analysis)
```
- **alg_json**
```
알고리즘 JSON
```
- **algorithm_code**
```
알고리즘 코드

1(Tensorflow)
2(Tensorflow-Keras)
4(Gensim)
5(Scikit-Learn)
```
### Return Type
```
success : "1"
error : "error"
```

<a name="GetAlgorithmInfo"></a>
# **GetAlgorithmInfo**
알고리즘 정보 반환(알고리즘 ID, 알고리즘명, 알고리즘 종류)
### Parameters
```
-
```
### Return Type
```
success : "1"
error : "error"
```

<a name="GetParamInfo"></a>
# **GetParamInfo**
알고리즘 ID를 통해 알고리즘 파라미터 상세 정보 반환(알고리즘 ID, 파라미터 ID, 파라미터 이름 등)
### Parameters
```
?alg_id=(String)
```
- **alg_id**
```
알고리즘 ID
```
### Return Type
```
success : "1"
error : "error"
```

<a name="InsertMLParamInfo"></a>
# **InsertMLParamInfo**
알고리즘 추천 후 알고리즘 종류, 코드 등에 대한 상세 정보 추가
### Parameters
```
Body: [{"param_id": (String),
       "alg_id": (String),
       "alg_anal_id": (String),
       "dp_analysis_id": (String),
       "project_id": (String),
       "param_json": (JSON format String)}
      ]
```
- **param_id**
```
파라미터 ID
```
- **alg_id**
```
알고리즘 ID
```
- **alg_anal_id**
```
알고리즘 분석 ID
```
- **dp_analysis_id**
```
데이터 분석 처리 ID
```
- **project_id**
```
프로젝트 ID
```
- **param_json**
```
파라미터 JSON
```
### Return Type
```
success : "1"
error : "error"
```