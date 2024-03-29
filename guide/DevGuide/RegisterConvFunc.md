# 변환함수 등록방법


새로운 유형에 대응할 수 있도록 변환 함수를 추가할 수 있으며, 그에 관한 방법을 제공하고자 함


# 클래스 파일 생성


- 클래스 파일은 [ConvertAbstract.py] 를 상속받아서 생성함
    - 경로 : AutoAPE-dataconverter/dataconverter/core/ConvertAbstract.py
- 파이썬의 파일명과 Class명을 일치하도록 작성
- 작성 완료한 class는 Autoape-dataconverter/dataconverter/core/functions/ 경로에 위치
    - 예시 : Autoape-dataconverter/dataconverter/core/functions/ExamlpleNewClassFile.py
- 변환작업은 전체 데이터가 아닌 한줄씩을 대상으로 하기 때문에, 변환을 수행하는 apply 함수에 한줄씩 입력되도록 작성
- apply 함수의 return 값은 list 자료형
    

    
- [ConvertAbstract.py] 코드 중 일부

```markdown
class ConvertAbstract(object):
    def __init__(
        self,
        arg_list: list = [],
        stat_dict: dict = {},
        logger: Logger = logging.getLogger(),
    ):
        self.num_feat = 1
        self.return_type = Constants.RETURN_TYPE_INT
        self.LOGGER = logger
        self.stat_dict = stat_dict
        self.arg_list = arg_list

        self.split_separator = ","
        self.max_len = 50
        self.padding_val = 255

	# Override 대상 함수
	# def processConvert(self, data):
	# def apply(self, data) -> list:
	# def get_num_feat(self) -> int:
	# def reverse(self, data, original_data):
	# def get_original_idx(self, cvt_data, original_data):
```


- [ConvertAbstract.py]를 상속 받아 생성한 변환함수 클래스 파일 코드 예시

```markdown
from dataconverter.core.ConvertAbstract import ConvertAbstract

class IfNull(ConvertAbstract):

    # 필수 재정의 함수
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self._replace = self.arg_list[0]
        self.num_feat = 1
        self.return_type = Constants.RETURN_TYPE_STRING

    # 필수 재정의 함수
    def apply(self, data: Any) -> list:

        # check blank
        if self._isBlank(data):
            return [self._replace]
        else:
            return [data]

	# 생략 가능 함수
	# def processConvert(self, data):
	# def apply(self, data) -> list:
	# def get_num_feat(self) -> int:
	# def reverse(self, data, original_data):
	# def get_original_idx(self, cvt_data, original_data):

if __name__ == "__main__":
    _str = ""
    print(IfNull(stat_dict=None, arg_list=["A"]).apply(_str))
```

# 변환함수 정보 DB 등록



- Back-End 데이터베이스 VAR_FUNC_INFO테이블에 작성한 클래스 정보를 추가
- 데이터베이스 관리 프로그램을 이용하여 데이터를 직접 추가하는 방식

| 컬럼명 | 작성방법 | 상황 | 입력 예시 |
| --- | --- | --- | --- |
| conv_func_id | Max(conv_func_id) + 1 | 컬럼내 가장 큰 값이 71인 경우 | 72 |
| conv_func_nm | 파일명, 클래스명과 동일 | 파일명이 IfNull 인 경우 | IfNull |
| conv_func_cls | 파일명, 클래스명과 동일 | 파일명이 IfNull 인 경우 | IfNull |
| conv_index | Max(conv_index) + 1 | 컬럼내 가장 큰 값이 65인 경우 | 66 |
| conv_func_tag | [[@카멜케이스클래스명()]] | 클래스명이 IfNull 인 경우 | [[@if_null()]] |
| conv_func_param_cnt | 매개변수 수량 입력 | 매개변수가 0개인 경우 | 0 |
| conv_func_cont | 변환 함수의 역할을 설명 | - | Null을 변환하는 기능을 한다. |
| proc_id | 작성자 id입력 | 작성자 id가 admin인 경우 | admin |
| proc_dt | 작성연월일시분 입력 | 23년 성탄절 오후 2시10분 | 202312251410 |
| user_made_yn | 항상 N 입력 | - | N |

# 이미지 빌드



- 신규 함수를 적용하기 위해서, dataconverter를 의존하고 있는(dependency) DA, MLPS의 모듈 이미지를 빌드하고 레지스트리에 PUSH 해주어야 함
- AutoAPE-mrms/conf/**mrms-conf.xml** 파일에 신규 이미지 tag 적용

```markdown
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl"  href="configuration.xsl"?>
<configuration>

	<property>
		<name>da.tag</name>
		<value>4.0-2023.3.0901</value>
		<description>DA Module Version</description>
	</property>

</configuration>
```
