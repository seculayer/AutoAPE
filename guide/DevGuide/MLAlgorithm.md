# ML 알고리즘 개발

#### 신규 알고리즘 추가 방법을 제공하고자함

# 클래스 파일 생성

- 클래스 파일 생성
	- 클래스 파일은 AlgorithmAbstract을 상속받아 작성하여 생성함
	- 경로 : AutoAPE-apeflow/apeflow/api/algorithms
	- 작성하는 알고리즘이 경로에 있는 라이브러리를 사용할 경우 해당 Abstract를 상속받아 작성(예 : tensorflow-keras를 이용한 알고리즘일 경우 AutoAPE-apeflow/apeflow/api/algorithms/tf/keras/TFKerasAlgAbstract를 상속받음)
	- 파일 작성 후 apeflow 모듈 내 경로(AutoAPE-apeflow/apeflow/api/algorithms/[해당 라이브러리])에 위치시킨다.


### 다음은 생성하는 python 파일 구조에 대해 설명하겠다. 

- [AlgorithmAbstarct.py] 코드

```python
class AlgorithmAbstract(object):
	ALG_CODE = "AlgorithmAbstract"
	ALG_TYPE = []
	DATA_TYPE = []
	VERSION = "1.0.0"
	LIB_TYPE = "None"

	def __init__(self, param_dict: dict, wrapper=None, ext_data=None):
		# 파라미터 체크
		_param_dict = dict(param_dict, **param_dict.get("params"))
		self.ext_data = dict() if ext_data is None else ext_data
		if wrapper is None:
			self.param_dict = self._check_parameter(_param_dict)
			self.learn_params = self._check_learning_parameter(_param_dict)
		else:
			self.param_dict = _param_dict
			self.learn_params = _param_dict
		self.early_steps = 0
		self.batch_size = Constants.BATCH_SIZE

		self.model = None
		try:
			self.task_idx = int(json.loads(os.environ["TF_CONFIG"])["task"]["index"])
		except Exception as e:
			self.task_idx = 0

	@staticmethod
	def _check_parameter(param_dict):
		_param_dict = dict()

		...

		return _param_dict

	def _check_learning_parameter(self, param_dict):
		_param_dict = dict()
	
		...

		return _param_dict()

	def early_stop(self, **kwargs):

		...

	
	def learn(self, dataset: dcit():
		raise NotImplementedError

	def predict(self, x) -> Dict:
		
		...

		return {"pred": results_pred, "proba": results_proba}

```
모델에 필요한 파라미터를 체크하고 설정한다.
_check_parameter는 모델 설정을 담당하는 부분이고, _check_learning_parameter는 모델 학습 설정을 담당하는 부분이다.
- _check_parameter : input_units, output_unints, model_nm, alg_sn, global_sn, algorithm_type, job_key, learning 등 모델에 필요한 파라미터를 설정한다.
- _check_learning_parameter : global_step(epochs), early_type(early stop 지정) 등 학습에 필요한 파라미터를 설정한다.

### AlgorithmAbstract을 상속받는 모델 클래스의 구조

- [LightGBM.py] 코드 구조

```python
class LightGBM(AlgorithmAbstract):
	# MODEL INFORMATION
	ALG_CODE = "LightGBM"
	ALG_TYPE = ["Classifier"]
	DATA_TYPE = ["Single"]
	VERSION = "1.0.0"
	OUT_MODEL_TYPE = Constants.OUT_MODEL_LGBM
	LIB_TYPE = Constants.GPU_SINGLE

	def __init__(self, param_dict, wrapper=None, ext_data=None):
		self.model: Union[LGBMClassifier, None] = None
		super(LightGBM, self).__init__(param_dict, wrapper=wrapper, ext_data=ext_data)

		if wrapper is None:
			self._build()
		else:
			self.load_model()


	def _check_parameter(self, param_dict):
		_param_dict = super(LightGBM, self)._check_parameter(param_dict)
		try:
    		_param_dict["num_leaves"] = int(param_dict.get("num_leaves", 41))
    		_param_dict["max_depth"] = int(param_dict.get("max_depth", 21))
		except Exception as e:
			raise ParameterError
		return _param_dict

	def _build(self):
		self.model = LGBMClassifier(
            num_leaves=self.param_dict["num_leaves"],
			max_depth=self.param_dict["max_depth"],
			objective="binary"
		)

	def learn(self, dataset:dict):
		global_step = self.learn_params["global_step"]
		global_sn = self.param_dict["global_sn"]

		result_callback = LearnResultCallback(
			global_sn=global_sn,
			job_key=self.param_dict["job_key"],
			epochs=global_step,
			task_idx=self.task_idx,
			data_len=len(dataset.get("x"))
		)

		self.model.fit(
			dataset.get("x"), self._arg_max(dataset.get("y")),
			eval_set=(dataset.get("x"), self._arg_max(dataset.get("y"))),
			callbacks=[result_callback.eval_callback()]
		)

	def load_model(self):
		TFSaveModel.load(self)
	
	def saved_model(self):
		TFSaveModel.save(self)

def eval_callback():
	def _callback(env : lightgbm.callback.CallbackEnv):
		print(env.iteration, env.evaluation_result_list)
	return _callback


```
해당 모델을 생성하고 학습할 수 있도록 파라미터를 세팅한다.


Tensorflow-keras 모델은 별도의 Abstract를 사용하여 해당 Abstract에서 AlgorithmAbstract을 상속받는다.

- [TFKerasAlgAbstract.py] 코드 구조


```python
class TFKerasAlgAbstract(AlgorithmAbstract):
	ALG_CODE = "TFKerasAlgAbstract"
	ALG_TYPE = []
	DATA_TYPE = []
	VERSION = "2.0.0"
	LIB_TYPE = Constants.KERAS
	TAG = "serve"
	OUT_MODEL_TYPE = Constants.OUT_MODEL_TF

	def __init__(self, param_dict, wrapper=None, ext_data=None):
		AlgorithmAbstract.__init__(self, param_dict, wrapper, ext_data=ext_data)

		self.num_workers = param_dict["num_workers"]
		
		self.input_name = "{}_{}_inputs".format(param_dict["model_nm"], param_dict["alg_sn"])
		self.output_name = "{}_{}_predicts".format(param_dict["model_nm"], param_dict["alg_sn"]

		self.model = None

		...

		if wrapper is None:
			self._build()
		else:
			self.load_model()
	
	def _build(self):
		raise NotImplementedError

	def _make_train_dataset(self, data):
		
		...
		
		return dataset, parallel_step

	def saved_model(self):
		TFSaveModel.save(self)

	def load_model(self):
		TFSaveModel.load(self)

	def learn(self, dataset):
		...

	def predict_decision(self, batch_x):
		batch_x = tf.keras.backend.cast(batch_x, tf.float32)
		return self.model(batch_x).numpy()

	def predict(self, x):

		...

		return {"pred": results_pred, "proba": results_proba}

	def split_data(self, dataset, l_ratio=80) -> Tuple[dict, dict]:
		l_dataset = dict()
		v_dataset = dict()
		len_data = len(dataset['x'])


		...

		return l_dataset, v_dataset

		
```


### TFKerasAlgAbstract을 상속받는 모델 클래스의 구조
- [KCNN.py] 코드 구조

```python
class KCNN(TFKerasAlgAbstract):
	#MODEL INFORMATION
	ALG_CODE = "KCNN"
	ALG_TYPE = ["Classifier", "Regressor"]
	DATA_TYPE = ["Single"]
	VERSION = "1.0.0"

	def __init__(self, param_dict, wrapper=None, ext_data=None):
		super(KCNN, self).__init__(param_dict, wrapper, ext_data)

	def _check_parameter(self, param_dict):
		_param_dict = super(KCNN, self)._check_parameter(param_dict)
		
		...

		return _param_dict

	def _build(self):
	
		...

		self.model = tf.keras.Sequential()
		self.inputs = tf.keras.Input(shape=input_units, name="{}_{}_x".format(model_nm, alg_sn))
		self.model.add(self.inputs)

		...

		self.model.summary(print_fn=self.LOGGER.info)

```
--------------------

## ML 알고리즘 정보 DB 등록

- Back-End DB ALGORITHM_INFO 테이블에 해당 클래스에 대한 정보 추가

| 컬럼명  | 작성방법    | 예시  | 설명  |
|:----------|:----------|:----------|:----------|
| alg_id            | lib_type + 0 + 알고리즘 등록 순서    |   20000000000000001   | lib_type이 Tensorflow-Keras이며 첫번째 등록된 알고리즘 |
| alg_nm            | 알고리즘 이름    | K-DNN    | Keras모델로 작성한 DNN 알고리즘    |
| algorithm_code    | alg_nm, 클래스 파일이름과 동일하게 작성    | KDNN    | KDNN.py로 작성된 알고리즘    |
| alg_type          | 성alg_type 표에 맞게 작성    | 1,2    | Classifier와 Regressor에서 사용 가능    |
| dist_yn           | Y/N 작성  |  Y   | 해당 알고리즘의 분산 가능 여부 |
| alg_cont          | 알고리즘 풀네임  | Keras Deep Neural Network    | alg_nm을 풀어서 작성    |
| alg_ver           | 알고리즘 버전    |  1.0   | - |
| proc_id           | 알고리즘 등록자    | admin    | 알고리즘 등록자를 작성    |
| proc_dt           | 등록일시[yyyyMMddHHmm]    | 201905200900    | -    |
| lib_type          | lib_type 표에 맞게 작성    | 2    | 해당 알고리즘의 라이브러리 번호롤 입력    |
| user_made_yn      | 항상 N으로 작성    |   N  |-|


###### alg_type 표

| alg_type	| description 
|:----------|:----------
| 1	| Classifier 
| 2	| Regressor 
| 3	| Clustering
| 4	| Word Embedding
| 6	| Feature Extraction
| 7	| Outlier Detection
| 10 |	Trend Analysis

###### lib_type 표

| lib_type	| description
|:----------|:----------
| 1	| Tensorflow
| 2	| Tensorflow-Keras
| 4	| Gensim
| 5	| Scikit-Learn


--------------------

## Docker 이미지 빌드

- 신규 알고리즘을 적용하기 위해 MLPS 모듈 이미지를 빌드하고 registry에 PUSH 해주어야함
- MLMS configmap에 신규 이미지 tag 적용 / PUSH 해준 이미지의 태그로 변경한다.
	- 경로 : AutoAPE-mrms/conf/mrms-conf.xml


```markdown
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl"  href="configuration.xsl"?>

   ...

    <property>
      <name>mlps.tag</name>
      <value>4.0-2023.3.0901</value>
      <description>MLPS Module Version</description>
    </property>

```
















































