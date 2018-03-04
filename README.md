Get Crypto Dataset from Google BigQuery Public Dataset
--------------

일반적으로 Gemini, Coinbase, Korbit등 여러 Exchange에서 Bitcoin 시세를 가져올 수 있는 API를 제공하고 있고 Quandl 같은 곳에서 유료로 가격 변동 정보를 다운 받을 수도 있다. 방법이 있다는건 알지만 시도는 해보지않고 있었는데, 우연히 Kaggle Datasets에서  비트코인 가격정보 BigQuery 데이터셋을 발견해서 빠르게 시도 해 볼 수 있을 것 같아 테스트를 해봤다.  

Google BigQuery는 서버리스 분석용 데이터웨어하우스로 SQL 쿼리, Spark, Hadoop을 사용해서 데이터를 빠르게 읽고 쓸 수 있다. 일반적으로 Kaggle에서 제공하는 데이터셋은 파일형식이고 그래서 인지 데이터셋 용량도 대부분 MB단위이다. 그에 반해 BigQuery로 제공되는 데이터셋은 GB수준 이상이고 데이터 내용도 주기적으로 업데이트 된다는 장점이 있다. 데이터에 접근하기 위해서는 python 클라이언트를 사용해야 하고 select query기능만 사용 할 수 있다.  

## Getting Started 

우선, BigQuery 데이터셋을 활용하는 방법 부터 알아보자. 30일 동안 5TB 쿼터가 있어 다시 사용하려면 새로 30일이 될때까지 기다려야 한다. 이런 불상사를 피하기 위해서 공식적으로 안내된 best practices를 지키는게 좋겠다. 

1. Authentification settings 
	- GCP 서비스 계정이 없다면 Authentification 설정이 필요하다. 우선적으로 서비스 계정 키를 만들어야 한다. 여기서 새 서비스 계정(계정이름-맘대로, 역할-소유자)을 만든다음 json파일을 다운 받는다. json파일 자체가 credentials이므로 아래처럼 설정 해준다	

	```
	vim ~/.bash_profile
	export $GOOGLE_APPLICATION_CREDENTIALS="Path/to/your/json/file"
	source ~/.bash_profile
	```

2. BigQuery Python Client Library 
	- GCP 공식 repository에서 최신 requirements 를 반영해서 라이브러리를 설치한다

	```
	git clone https://github.com/GoogleCloudPlatform/python-docs-samples
	cd python-docs-samples/bigquery/cloud-client
	pip install -r requirements.txt
	```

## Usage 

우선 	python에서 아래와 같이 BigQuery 클라이언트 로드를 테스트한다 


	```
	from google.cloud import bigquery
	client = bigquery.Client()
	```

Google BigQuery 에서 제공하는 Public Dataset중 에도 여러 프로젝트가 있다. 여기서 우리는 `bitcoin_blockchain`프로젝트를 사용할 예정이다. 프로젝트에는 아래와 같이 8개 테이블이 있다. 

	- bigquery-public-data.bitcoin_blockchain.bitfinexUSD
	- bigquery-public-data.bitcoin_blockchain.bitstampUSD
	- bigquery-public-data.bitcoin_blockchain.blocks
	- bigquery-public-data.bitcoin_blockchain.coinbaseUSD
	- bigquery-public-data.bitcoin_blockchain.hitbtcUSD
	- bigquery-public-data.bitcoin_blockchain.krakenUSD
	- bigquery-public-data.bitcoin_blockchain.mtgoxUSD
	- bigquery-public-data.bitcoin_blockchain.transactions

예시 쿼리 코드 실행 

```
# Query by Allen Day, GooglCloud Developer Advocate (https://medium.com/@allenday)
query_job = """
#standardSQL
SELECT
  o.day,
  COUNT(DISTINCT(o.output_key)) AS recipients
FROM (
  SELECT
    TIMESTAMP_MILLIS((timestamp - MOD(timestamp,
          86400000))) AS day,
    output.output_pubkey_base58 AS output_key
  FROM
    `bigquery-public-data.bitcoin_blockchain.transactions`,
    UNNEST(outputs) AS output ) AS o
GROUP BY
  day
ORDER BY
  day
"""

query_job = client.query(query)

iterator = query_job.result(timeout=30)
rows = list(iterator)

# Transform the rows into a nice pandas dataframe
transactions = pd.DataFrame(data=[list(x.values()) for x in rows], columns=list(rows[0].keys()))

# Look at the first 10 headlines
transactions.head(10)
```