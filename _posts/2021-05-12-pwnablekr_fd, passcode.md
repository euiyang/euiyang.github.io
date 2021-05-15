# fd  
이 문제는 file descript를 입력받는 1번째 인자에 의해 결정이 되므로 fd를 표준 입력인 0을 입력해주기 위해 0x1234를 입력한다.  

# passcode  

GOT overwrite 공격기법:  
dynamic link 방식으로 컴파일된 바이너리가 공유 라이브러리를 호출할때 사용되는 PLT,GOT를 이용하는 공격 기법.  
PLT(procedure linkage table): 외부 프로시저를 연결해주는 테이블.  
GOT(global offset table): 프로시저들의 주소가 담겨있으며 ,PLT가 참조하는 테이블이다.  

PLT와 GOT 관계:  
![image](https://user-images.githubusercontent.com/65746019/118376160-4859cf80-b601-11eb-8f20-41d7a8ae0c85.png)  

dynamic link 방식으로 프로그램이 만들어지면 첫 호출의 경우 linker가 dl_resolve라는 함수를 사용해서 필요한 함수의 주소를 알아오고 GOT에 실제 함수의 주소가 써준다. 하지만 두번째 호출에서는
