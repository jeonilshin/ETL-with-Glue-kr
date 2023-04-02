## [핸즈-온] AWS Glue Studio로 ETL 작업 (New)
이번 포스팅은 Glue Studio로 반정형 데이터에 대한 ETL 작업을 해 보겠습니다.
# 0. 개요
**ETL**작업은 필요한 데이터를 만들기 위해 원본 데이터를 사용자의 목적에 맞도록 가공하는 작업입니다. **AWS Glue**는 단지 **ETL**작업만 지원하기 때문에, 이 **ETL**작업을 통해 어떤 데이터로 가공할 것 인지에 대해 생각해야 합니다.

이러한 ETL작업에서 가장 중요한 관점은 **“목적성”** 입니다. 사용자는 AI모델을 만들거나, 로그 분석을 통해 장애를 파악하는 등의 데이터가 필요한 “목적”이 있습니다. 이를 위해 ETL작업은 **“목적성”** 이 있어야 원하는 데이터로 가공을 할 수 있습니다.

이번 포스팅에서의 **Hands On**으로 <ins>AWS에서 Webina로 진행한 Glue – Sagemaker의 예를 Glue Studio로 수행</ins>하겠습니다.

우리에게는 예시 데이터로 아마존 쇼핑몰의 리뷰 데이터가 있다고 가정합시다. 이 데이터를 가지고만 있으면 아무것도 아닌, 단순한 리뷰 데이터입니다.

만약 우리에게 아래와 같은 과제가 주어졌다고 합시다.
![image](https://user-images.githubusercontent.com/86287920/229339759-0a853f73-560e-49a7-bf7a-bd63e503f000.png)

우리에게는 머신러닝 모델을 만들 수 있는, 즉 **“목적”** 을 달성할 수 있는 데이터가 필요합니다. 다시말하면, 우리가 가지고 있는 아마존 쇼핑몰의 리뷰 데이터를 가지고 머신러닝 모델을 만들 **“목적성”** 을 가질 수 있게 됩니다.

이번 포스팅에서 **“목적성”** 을 달성하기 위해 우리는 가지고 있는 아마존 쇼핑몰 리뷰 데이터를 가지고 머신러닝을 수행할 수 있는 데이터로 ETL작업을 Hands On을 통해 진행하겠습니다.

Hands On을 진행하기 위해서는 쇼핑몰의 리뷰에 해당하는 데이터가 필요합니다. Amazon에서는 이러한 모델을 만드는 것을 지원하기 위해 Amazon 쇼핑몰의 review데이터를 제공합니다.

링크 : [https://s3.amazonaws.com/amazon-reviews-pds/readme.html](https://s3.amazonaws.com/amazon-reviews-pds/readme.html)

Hands On의 ETL 작업은 다음과 같은 순서로 이루어 집니다.

- Data Crawling
- Data Analyzing
- Data Partitioning
- Data Balancing

Hands On에서는 원활한 테스트를 위해 Amazon Review 데이터의 sample_us.tsv 파일을 이용합니다.

또한 여러 카테고리에 대한 Partitioning을 테스트하기 위해 sample_us.tsv파일을 임의 복제 하였습니다. 다음의 사진은 임의 복제한 파일의 데이터 수 및 schema 입니다.
![image](https://user-images.githubusercontent.com/86287920/229340691-b9e4b450-ea29-43bb-8488-446986cb62a5.png)
Figure 1) sample_us.tsv를 임의 복제한 데이터, 스키마(좌)와 크기(우)

# 1. Data Crawling
Glue에서는 Data를 직접 가져오는 것이 아닌, Data의 메타데이터만을 가져와서 데이터 카탈로그에 저장합니다.

**<메타데이터와 데이터 카탈로그는 무엇일까요?>**

**메타데이터**란, 데이터의 특성을 나타낸 것으로 크롤링된 데이터의 크기, 데이터의 형식, 스키마 등등으로 원본 데이터를 표현한 데이터입니다.

**데이터 카탈로그**는 이러한 메타데이터 들을 모아 한눈에 여러 데이터들을 확인할 수 있도록 해주는 기능입니다. Glue 에서는 “테이블”이라는 이름으로 데이터 카탈로그가 존재하며, 저장된 모든 메타데이터 들을 확인할 수 있으며 스키마 수정 등 간단한 메타데이터에 대한 작업을 할 수 있습니다.

작업을 할 때마다 모든 데이터를 가져오게 된다면 어떻게 될까요? 데이터의 양이 MB수준이면 가능하겠지만, GB, TB수준까지만 올라도 매우 큰 네트워크 트래픽이 발생할 것 입니다. 이를 방지하기 위해, 최후의 데이터 가공까지 원본 데이터를 옮기지 않습니다.(DB의 Lazy Evaluation과 비슷한 맥락입니다.)

Data의 메타데이터를 가져오는 작업은 Glue Crawler를 통해 할 수 있습니다.

![image](https://user-images.githubusercontent.com/86287920/229341227-ea89756e-ef68-4bd5-b3b7-9e368f10c520.png)

[AWS Glue] -> [크롤러] -> [크롤러 추가]

![image](https://user-images.githubusercontent.com/86287920/229341315-bcff7768-a83d-47a5-b7ed-b31146568ebe.png)
![image](https://user-images.githubusercontent.com/86287920/229341410-0bdc5fda-43f2-41f3-8181-5f066b579d10.png)

