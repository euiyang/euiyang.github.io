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


