# 드림핵 3번 문제  
![image](https://user-images.githubusercontent.com/65746019/117529038-3fb93600-b010-11eb-9b4d-31f7e41f3099.png)  
1번 부분: eax에는 반복문에서 배열의 인덱스를 의미한다.  
2번 부분: cmp eax(index),18 이므로 0x18번 반복을 한다는 의미이다.  
3번 부분: 정답 배열을 rcx에 넣고 rax(인덱스)를 더해서 해당하는 문자를 eax에 넣는다.  
4번 부분: 내가 넣은 정답 부분을 암호화하는 과정이다. 우선 인덱스를 rcx에 넣은 후 인자로 받은 배열을 rdx에 넣는다. rdx(배열)+rcx(인덱스)가 가르키는 곳이 현재 비교할 부분이고 이 값(ecx)을 암호화한다.   

암호화:  
ecx를 인덱스와 일단 xor 연산을 한 뒤 index * 2를 더하는 방법을 사용해서 암호화 했다.  
-> ecx^i + i*2  

복호화:  
1)복호화 하기 위해 암호화 하는 값을 가져오는데 가져올 곳은 3번 부분에서 정답을 가져왔으므로 저 쪽가서 24bytes만큼 가져온다.  
2)암호화의 반대로 진행하기 위해 암호화의 역과정을 거치면 된다.
3)(암호화 된 부분- i*2)^i을 반복


![image](https://user-images.githubusercontent.com/65746019/117529329-f964d680-b011-11eb-9f4f-b0a9b576a92c.png)  



# IAT(import address table)  
어떤 라이브러리에서 어떤 함수를 사용하는지 적은 테이블이다.  

---

## DLL(dynamic link library)  
요즘에는 멀티 테스킹을 지원하기 때문에 여러 프로그램에서 동일한 라이브러리가 실행되어야하기 때문에 비효율적이다.  
DLL은 직접 프로그램에 lib를 포함시키지 말고 DLL을 만들어서 거기에 연결시켜서 여러 프로세스에서 공유하자.  

DLL 로딩방식
1)explicit linkng: 프로그램 사용되는 순간에 로딩하고 사용이 끝나면 해제  
2)implicit linking: 프로그램 시작과 동시에 로딩 후 프로그램 종료시 해제  

---

IAT는 두가지 방법 중에서 implicit linking에 대한 메커니즘을 제공한다.  

# image_import_descriptor  
자신이 어떤 lib를 import하는지 명시된 곳.  
![image](https://user-images.githubusercontent.com/65746019/117534888-42c31f00-b02e-11eb-88a5-d6c8c646bde5.png)  

프로그램은 여러개의 lib를 가져오기 때문에 구조체의 배열형식으로 존재하고 마지막은 NULL로 표시한다.  
중요한 세 개만 보자  
1) originalfirstthunk: import name table(INT)의 RVA. 여기서 table은 배열을 의미.  
2) name: lib 이름 문자열의 RVA  
3) firstthunk: import address table(IAT)의 RVA  

pe loder가 가져오는 함수의 주소를 IAT에 입력하는 순서  
1)lib 이름 문자열 얻고 로딩  
2)image_import_descriptor(IID)에 originalfirstthunk 멤버를 읽어서 INT주소얻고 내부의 배열의 값을 읽어 image_by_name 주소를 얻음  
3)name 항목을 통해 해당 함수의 시작 주소를 얻음.  
4)IID의 firstthunk(IAT)멤버를 읽어서 IAT 주소를 얻음
5)IAT 배열 값에 4번에서 구한 IAT 주소를 입력  
6)끝날 때까지 반복

실습은 나중에 해보자  


# EAT  
lib 파일에서 제공하는 함수를 다른 프로그램에서 쓸 수 있도록 만드는 메커니즘.  
![image](https://user-images.githubusercontent.com/65746019/117535796-553f5780-b032-11eb-89c0-4bce21eac2cd.png)

1)number of functions: 실제 export 함수 개수  
2)number of names: export 함수 중 이름을 가지는 함수 개수  
3)address of functions: export 함수 주소 배열  
4)address of names: 함수 이름 주소 배열  
5)address of name ordinals: ordinal 주소 배열  

GetProcAddress 함수의 동작 원리  
1)address of names 멤버를 이용해 함수 이름 배열로 가서 strcmp를 통해 원하는 함수 찾음  
2)address of name ordinal 멤버를 통해 ordinal 배열로 가고 name_ index를 통해 ordinal 값을 찾음.  
3)address of functions 멤버로 함수 주소 배열(EAT)로 간다.  
4)ordinal과 EAT로 함수 시작 주소를 얻는다.  

이거도 실습 나중에  




# 실행 압축  
데이터 압축: 
1) 비손실 압축: 데이터의 무결성을 보장하고 압축하는 것, 대표적인 비손실 압축은 run-length,lempel-ziv,huffman 방법이 있다.  
2) 손실 압축: 데이터에 의도적인 손상을 주고 압축률을 높이는 방법.  

실행 압축: PE 파일을 대상으로 압축해제 코드를 사용해서 실행시 메모리에 올라가서 압축이 해제되어 실행된다.  

일반 압축은 모든 파일 대상으로 높은 압축이 가능하지만 별도의 압축해제 프로그램이 필요하다.  
반면에 실행압축은 자체적으로 압축해제를 하기에 실행 시간이 조금 느리지만 pe 파일의 실행이 가능하다

패커(packer): PE 파일을 실행 압축 파일로 만들어주는 역할.  
패커의 목적  
1)크기 줄이기  
2)pe 파일의 내부 코드와 리소스 감추기 위해  

패커의 종류:
1) 평범한 pe 파일 생성 패커  
2) 원본 파일 변형과 pe header 망치는 패커


프로텍터: pe 파일을 리버싱으로부터 보호하기 위한 것으로 여러개의 프로텍터가 존재해서 압축 pe 파일이 원본보다 커지는 경우도 있다.  

프로텍터 사용 목적:  
1) 크래킹 방지
2) 코드나 리소스 보호  
3) 악성코드(trojan, worm): anti-virus 제품이 탐지하지 못하도록  

실습 나중에  










