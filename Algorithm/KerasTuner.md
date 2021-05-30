# Keras Tuner

딥러닝에 입문후 모델을 가져다 사용하다가 이제 어느정도 모델을 구성할 수 있다면, 어떻게 이 모델의 성능을 높일까 고민하게 된다. 최적의 모델을 위해 모델의 요소 값을 찾는 과정을 "하이퍼파라미터 튜닝"이라고 한다. 

한 모델의 하이퍼파라미터 요소의 개수가 많을수도 적을 수도 있어 모델 구성에 따라 성능의 차이는 천지차이이며 어렵다. 그래서 구글이 이러한 캐라스 모델을 쉽게 튜닝하기 위해 "Keras Tuner" 프레임 워크를 개발했다. 

[Cutting Edge TensorFlow: New Techniques (Google I/O'19)](https://youtu.be/Un0JDL3i5Hg)

"Keras Tuner"에 관한 발표 - 7분 20초 까지

공식 문서 기준으로 부분부분 이해해 보자. - [https://keras-team.github.io/keras-tuner](https://keras-team.github.io/keras-tuner/)

기본적으로 "Keras Tuner"을 사용하기 위해선 

Python 3.6(Version)

TensorFlow 2.0(Version)

"Keras Tuner"를 다운받아 줍니다.

```python
pip install -U keras-tuner
```

소스를 다운받아 줍니다.

```python
git clone https://github.com/keras-team/keras-tuner.git
cd keras-tuner
pip install .
```

처음으로  모델을 정의합니다.

```python
from tensorflow import keras
from tensorflow.keras import layers

def build_model(hp):
    model = keras.Sequential()
		# Dense Layer에 unit수 선택
    # 정수형 타입 32부터 512까지 32배수 범위 내에서 탐색
		# activation 은 relu 사용
    model.add(layers.Dense(units=hp.Int('units',
                                        min_value=32,
                                        max_value=512,
                                        step=32),
                           activation='relu'))

    model.add(layers.Dense(10, activation='softmax'))
    model.compile(
        optimizer=keras.optimizers.Adam(
		# 학습률은 자주 쓰이는 0.01, 0.001, 0.0001 3개의 값 중 탐색
            hp.Choice('learning_rate',
                      values=[1e-2, 1e-3, 1e-4])),
        loss='sparse_categorical_crossentropy',
        metrics=['accuracy'])
    return model
```

```python
# RandomSearch
from kerastuner.tuners import RandomSearch

tuner = RandomSearch(
    build_model, # HyperModel
    objective='val_accuracy', #  최적화할 하이퍼모델
    max_trials=5, 
    executions_per_trial=3, # 각 모델별 학습 회수
    directory='my_dir', # 사용된 parameter 저장할 폴더
    project_name='helloworld') # 사용된 parameter 저장할 폴더
```

```python
# Hyperband
from kerastuner.tuners import Hyperband

tuner = kt.Hyperband(
		build_model,, # HyperModel
		objective ='val_accuracy', #  최적화할 하이퍼모델
		max_epochs =20, # 각 모델별 학습 회수
		factor = 3,    # 한 번에 훈련할 모델 수 결정 변수
		directory ='my_dir', # 사용된 parameter 저장할 폴더
		project_name ='helloworld') # 사용된 parameter 저장할 폴더
```

```python
# 작성한 Hypermodel 출력
tuner.search_space_summary()
```

```python
# tuner 학습
tuner.search(x, y,
             epochs=5,
             validation_data=(val_x, val_y))
```

```python
# 최고의 모델을 출력
models = tuner.get_best_models(num_models=2)
# 혹은 결과 출력
tuner.results_summary()
```