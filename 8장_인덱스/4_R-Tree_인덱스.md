## 4. R-Tree 인덱스
GPS 정보나 지도 서비스 등 2차원 좌표 값을 이용하는 데이터를 인덱싱하고 검색하는 목적의 인덱스이다. 기본적인 내부 매커니즘은 B-Tree와 흡사하지만, 인덱스를 구성하는 칼럼의 값이 1차원의 스칼라 값이 아닌 2차원의 공간 개념 값이라는 차이점이 있다.

### 4.1. 구조 및 특성
MySQL은 공간 정보의 저장 및 검색을 위해 여러 가지 기하학적 도형 정보를 관리할 수 있는 데이터 타입을 제공한다. 대표적으로 지원하는 데이터 타입은 아래와 같다.
![alt text](./img/image4_0.png)

- MBR(Minimum Bounding Rectangle) : 해당 도형을 감싸는 최소 크기의 사각형
![alt text](./img/image4_1.png)


![alt text](./img/image4_2.png)
![alt text](./img/image4_3.png)

### 4.2. R-Tree 인덱스의 용도
R-Tree는 MBR 정보를 이용해 B-Tree 형태로 인덱스를 구축하므로 Rectangle의 'R'과 B-Tree의 'Tree'를 섞어서 R-Tree라는 이름이 붙여졌으며, 공간 인덱스라고도 한다.

일반적으로 GPS 기준의 위도, 경도 좌표 저장에 주로 사용하고 CAD/CAM 소프트웨어 또는 회로 디자인 등 좌표 시스템에 기반을 둔 정보에 대해 모두 적용 가능하다.