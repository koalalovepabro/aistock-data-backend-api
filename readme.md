# 맞춤형 주식 포트폴리오 추천 웹서비스 구현 프로젝트
## 프로젝트 개요
국민 1인당 1주식 계좌시대가 된 현재.  
저금리 기조가 장기간 이어지면서 원금 손실이 가능한 고위험 투자임에도 불구하고 고수익을 쫓는 직접 투자자들이 늘어나고 있다.  
이러한 사회경향을 반영하여 개인의 주식 포트폴리오 구성을 데이터 기반으로 추천해주는 프로젝트를 기획하였다.  <br><br>
본 프로젝트는 투자 금액에 맞추어 원하는 종목으로 구성된 포트폴리오를 최적화하는 알고리즘과 웹서비스를 구현한다.  
주로 고액 자산가만을 대상으로 하여 접근성이 낮았던 기존 개인 자산관리(PB) 서비스를 <b>소액의 투자금액</b>으로 <b>내가 원하는 시간과 장소</b>에서 <b>데이터 기반의 객관적인 추천</b>을 받을 수 있다는 것이 큰 장점이다.<br><br>
![image](https://user-images.githubusercontent.com/81539406/147208886-4bd5fd34-d73d-4cf6-9460-e8ace511b525.png)

## 설계의 주안점
1. 투자금액, 투자기간, 리스크(변동성) 제약, 투자 종목에 따라 <b>맞춤형</b> 포트폴리오 추천
2. 투자종목은 직접 선택할 수 있으며, <b>종목을 선택하는 5가지 전략</b>을 제시<br> 
   1) 모멘텀 1개월: 최근 1개월 수익률이 가장 높은 상위 30개 종목을 선택 (상대모멘텀)
   2) 모멘텀 3개월: 최근 3개월 수익률이 가장 높은 상위 30개 종목을 선택 (상대모멘텀)
   3) 듀얼모멘텀: 상대 모멘텀 x 절대 모멘텀
   4) 급등주: 특정 거래일의 거래량이 이전 20일의 평균 거래량보다 1,000% 이상 급증하는 종목을 선택
   5) 상승일 빈도: 최근 1년의 데이터를 기준으로, 전일 기준 상승 빈도가 높은 상위 30개 종목을 선택

3. <b>포트폴리오 최적화</b> 방법은 2가지가 있으며, 선택 가능
   1) Maximize 샤프지수
   2) 설정한 변동성 내에서 Maximize 수익률
4. 소액의 직접투자자도 부담 없이 이용 가능한 <b>개인 맞춤형 추천 서비스</b>
5. 시간과 장소 불문하고 편리하게 활용할 수 있는 <b>웹 서비스</b> 제공

## 기능 개요
1. 일, 특정 시간 단위의 스크립트 실행
2. 포트폴리오 만들기 시 결과를 api 로 리턴
3. 메인웹의 mysql에 데이터를 배치 처리

## 시연영상
[Project Demo Video](https://youtu.be/27oo2CBSu_w)

## 설치되는 패키지

패키지 목록
1. `pip install PyPortfolioOpt` : cvxpy, numpy, pandas, scipy
   1. `pip install cvxopt` : PyPortfolioOpt 에서 필요로 함.
2. `pip install matplotlib` : cycler, kiwisolver, numpy, pillow, pyparsing, python-dateutil
3. ~`pip install scikit-learn`~ : 
4. `pip install finance-datareader` : lxml, pandas, requests, requests-file, tqdm
   1. `pip install beautifulsoup4` : soupsieve
       - finance-datareader 에서 필요로 함.
5. `pip install pykrx` : datetime, deprecated, numpy, pandas, requests, xlrd
6. `pip install mysql-connector-python sqlalchemy`
   - mysql-connector-python : protobuf
   - sqlalchemy : greenlet
7. `pip install flask-restx` : ansio8601, Flask, jsonschema, pytz, six, werkzeug
   1. `pip install flask-cors` : flask, six
      - flask api를 ajax통신하는데 필요로 함.
8. `pip install tensorflow`
   - lstm 을 이용하는데 사용됨.


## Docker 생성 및 컨테이너 실행
```console
docker build --force-rm --no-cache -t aistock-stockdata-api .
docker run --env-file ./.local.env --name aistock-stockdata-api -d -p 26000:5000 -v "%cd%:/app" aistock-stockdata-api
```


기존에 만든 것이 있었다면 중간에 rm 을 먼저 한 번 해주고 실행한다.
```console
docker rm -f aistock-stockdata-api

docker build --force-rm --no-cache -t aistock-stockdata-api .
docker run --env-file ./.local.env --name aistock-stockdata-api -d -p 26000:5000 -v "%cd%:/app" aistock-stockdata-api
```

### 내부적인 코드
* /update_stock_table.py : 데이터베이스 테이블의 종목 목록을 갱신함
