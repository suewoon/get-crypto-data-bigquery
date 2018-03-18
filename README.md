Get Crypto Dataset from Google BigQuery Public Dataset
--------------

이 Repo에서는 Google BigQuery의 Public Dataset을 가져오는 방법과 Crypto Dataset을 활용한 [EDA 분석](https://nbviewer.jupyter.org/github/suewoon/get-crypto-data-bigquery/tree/master/google-big-query-test.ipynb)을 담고 있다.

## Getting Started 

1. Authentification settings 
	- GCP 서비스 계정이 없다면 Authentification 설정이 필요하다. 우선적으로 서비스 계정 키를 만들어야 한다. 여기서 새 서비스 계정(계정이름-맘대로, 역할-소유자)을 만든다음 json파일을 다운 받는다. json파일 자체가 credentials이므로 아래처럼 설정 해준다.	

	```
	vim ~/.bash_profile
	export $GOOGLE_APPLICATION_CREDENTIALS="Path/to/your/json/file"
	source ~/.bash_profile
	```

2. BigQuery Python Client Library 
	- GCP 공식 repository에서 최신 requirements 를 반영해서 라이브러리를 설치한다.

	```
	git clone https://github.com/GoogleCloudPlatform/python-docs-samples
	cd python-docs-samples/bigquery/cloud-client
	pip install -r requirements.txt
	```

## Usage 

우선 	python에서 아래와 같이 BigQuery 클라이언트 로드를 테스트한다. 

```
	from google.cloud import bigquery
	client = bigquery.Client()
```

Google BigQuery 에서 제공하는 Public Dataset중 에도 여러 프로젝트가 있다. 여기서 우리는 `bitcoin_blockchain`프로젝트를 사용한다.

예시

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