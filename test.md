# 파이썬 딥러닝 프로젝트에서 효과적인 로깅(Logging) 구현하기

## 1. 왜 print() 대신 logging을 사용해야 할까요?

딥러닝 프로젝트를 진행하면서 이런 고민들을 해보셨나요?
- "학습 중에 어떤 문제가 발생했는지 나중에 확인하고 싶은데..."
- "print()문을 여기저기 넣었더니 콘솔이 너무 지저분해졌어..."
- "학습 진행 상황을 파일로도 저장하고 싶은데..."
- "에러가 발생했을 때 어떤 데이터에서 문제가 있었는지 추적하고 싶은데..."

이런 문제들을 해결할 수 있는 것이 바로 Python의 logging 모듈입니다!

## 2. 로깅의 기본 개념 이해하기

### 2.1 로깅 컴포넌트
로깅 시스템은 크게 세 가지 주요 컴포넌트로 구성됩니다:

1. **Logger**: 로그 메시지를 생성하고 처리하는 주체
   ```python
   logger = logging.getLogger(__name__)
   ```

2. **Handler**: 로그 메시지를 어디로 출력할지 결정
   - FileHandler: 파일에 로그 저장
   - StreamHandler: 콘솔에 로그 출력
   - RotatingFileHandler: 파일 크기나 시간에 따라 로그 파일 순환

3. **Formatter**: 로그 메시지의 형식을 지정
   ```python
   formatter = logging.Formatter('%(asctime)s - %(levelname)s - %(message)s')
   ```

### 2.2 로깅 레벨
로깅은 5가지 기본 레벨을 제공합니다:

1. DEBUG (10): 상세한 디버깅 정보
2. INFO (20): 일반적인 정보
3. WARNING (30): 경고 메시지
4. ERROR (40): 오류 메시지
5. CRITICAL (50): 심각한 문제

```python
logger.debug("데이터 전처리 시작...")  # 개발 중 상세 정보
logger.info("모델 학습 시작")          # 일반적인 진행 상황
logger.warning("GPU 메모리 부족")      # 주의가 필요한 상황
logger.error("데이터 로딩 실패")       # 오류 발생
logger.critical("모델 저장 실패")      # 심각한 문제
```

## 3. 로깅 시스템 구현하기

### 3.1 기본 설정
프로젝트의 로깅 설정을 구현해보겠습니다:

```python
def setup_logging(experiment_folder):
    # 1. 루트 로거 생성 및 레벨 설정
    logger = logging.getLogger()
    logger.setLevel(logging.INFO)
    
    # 2. 파일 핸들러 설정
    file_handler = logging.FileHandler(f"{experiment_folder}/experiment.log")
    
    # 3. 콘솔 핸들러 설정
    console_handler = logging.StreamHandler(sys.stdout)
    
    # 4. 포매터 설정
    formatter = logging.Formatter(
        fmt='%(asctime)s - %(levelname)s - %(name)s - %(message)s',
        datefmt='%Y-%m-%d %H:%M:%S'
    )
    
    # 5. 핸들러에 포매터 적용
    file_handler.setFormatter(formatter)
    console_handler.setFormatter(formatter)
    
    # 6. 로거에 핸들러 추가
    logger.addHandler(file_handler)
    logger.addHandler(console_handler)
```

### 3.2 실제 프로젝트에서의 활용

#### 데이터 로딩 및 전처리 로깅
```python
def load_and_prepare_data(config):
    logger = logging.getLogger(__name__)
    try:
        logger.info(f"데이터 로딩 시작: {config.data_params.file_path}")
        df = pd.read_csv(config.data_params.file_path)
        logger.info(f"총 {len(df)}개의 데이터 로드 완료")
        
        # 전처리 과정 로깅
        logger.debug(f"데이터 형태: {df.shape}")
        logger.debug(f"결측치 현황: {df.isnull().sum().sum()}")
        
    except Exception as e:
        logger.error(f"데이터 로딩 중 오류 발생: {str(e)}")
        raise
```

#### 모델 학습 과정 로깅
```python
def train(model, train_loader, optimizer, criterion, config):
    logger = logging.getLogger(__name__)
    logger.info("학습 시작...")
    
    try:
        for epoch in range(config.training_params.max_epochs):
            loss = model.train_step()
            logger.info(f"Epoch {epoch}: loss = {loss:.4f}")
            
            if loss < best_loss:
                logger.info(f"새로운 최적 모델 저장 (loss: {loss:.4f})")
                
    except Exception as e:
        logger.error(f"학습 중 오류 발생: {str(e)}")
        logger.debug(f"현재 배치 데이터: {batch.shape}")
        raise
```

## 4. 로깅 모범 사례와 팁

### 4.1 효과적인 로그 메시지 작성
- 시간, 메트릭, 설정값 등 중요 정보 포함
- 문제 발생 시 원인 파악이 가능하도록 상세 정보 기록
- 로그 레벨 적절히 사용

### 4.2 성능 최적화
- DEBUG 레벨 로깅은 조건부로 사용
```python
if logger.isEnabledFor(logging.DEBUG):
    logger.debug(f"대용량 데이터 상태: {huge_data.describe()}")
```

- 로그 파일 관리를 위한 RotatingFileHandler 사용
```python
handler = RotatingFileHandler(
    'experiment.log',
    maxBytes=10*1024*1024,  # 10MB
    backupCount=5
)
```

### 4.3 실험 관리를 위한 로깅 전략
- 실험 설정, 하이퍼파라미터 기록
- 중간 결과 및 메트릭 저장
- 에러 발생 시 디버깅 정보 수집

## 5. 요약
- print() 대신 logging을 사용하면 레벨 구분, 파일 저장 등 다양한 기능 활용 가능
- 적절한 로깅 설정으로 실험 관리와 디버깅이 용이
- 로그 레벨과 포매터를 활용해 체계적인 로그 관리 가능

이제 여러분의 딥러닝 프로젝트에 로깅 시스템을 도입해보세요. 실험 관리가 한결 수월해질 거예요! 😊
