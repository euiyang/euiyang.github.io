# PE file  
windows에서 사용하는 실행 파일 형식.
종류는 크게 실행파일, obj파일로 구분할 수 있다.  

## PE 기본 구조  
DOS header부터 section header 까지를 PE body라 부른다.  
파일은 offset, 메모리에서는 VA(vertual memory)로 위치를 표현한다.  
파일 내용은 .text(코드),.data(데이터),.rsrc(리소스) 섹션에 나눠서 저장한다.  

섹션 헤더에 각 섹션마다의 파일/메모리에서의 크기, 위치, 속성이 정의되어 있으며, 각 섹션의 끝부분과 PE 헤더의 끝에는 Null 값으로 padding 되어 있다.  

***

# VA, RVA  
VA는 프로세스 가상 메모리의 절대주소를 말하고, RVA는 기준 위치로부터의 상대주소를 말한다.  
두 개의 관계는 RVA+ ImageBase= VA 가 성립한다.  
PE header는 많은 구조체로 이루어져 있고, 내부는 재배치 과정을 통해 빈 공간에 로딩되어야하기 때문에 RVA 주소로 되어 있다.  
![image](https://user-images.githubusercontent.com/65746019/116503047-1ef62f80-a8f0-11eb-9a39-060267910a4e.png)  

***

# image 뜻  
아래에 나오는 IMAGE_NT_HEADER 등에서 사용되는 image의 뜻은 메모리에 로딩된 파일과 아닌 파일을 구별하기 위해서 메모리에 로딩된 파일을 image라 부른다.  

***

# DOS header(notepad.exe를 hxd에서 분석)  
맨 처음 부분의  
e_magic: DOS signature(ascii: MZ)  
e_lfanew: NT header의 offset을 표시(파일에 따라 변함)  
을 알아야 한다. e_ magic은 파일의 시작부분에 DOS signature가 항상 존재하고, e_lfanew가 가리키는 위치에 NT Header가 존재해야 한다.  
![image](https://user-images.githubusercontent.com/65746019/116502820-8bbcfa00-a8ef-11eb-8545-60b2dd99b681.png)  

## DOS STUB
DOS header 밑에 존재하며 크기도 일정하지 않다. 이 DOS stub는 16비트 DOS 환경에서 동작하기 때문에 Windows에서는 무시한다.  
DOS header는 이전에 사용되는 DOS 파일에 대한 호환성을 고려해서 만들어진 공간이며 필요시 16비트의 DOS용 코드를 실행할 수 있다.(16비트 코드가 존재할 시.)  

***

# NT header  
구조체는 signature, file header, optional header로 구성되어 있다.  
![image](https://user-images.githubusercontent.com/65746019/116529854-5678d200-a918-11eb-9172-5c77a4e34d35.png)  


## NT header- PE signature  
PE signature의 값은 50450000으로 구조체 내부에 NT-header의 signature가 ascii 값으로 PE이기 때문에 시작 위치를 알 수 있다.  

## NT header- file header
file header에는 파일의 속성을 나타내는 구조체이다.  
이 구조체 내부에서 중요한 4가지 멤버인 machine, numberofsections, SizeOfOptionalHeader, characteristics를 알아보자.  

1)machine: CPU 별 고유한 값으로 intel x86 호환 칩은 14C(\x4c \x01)의 값을 가진다. 나는 amd CPU라서 그런지 8664가 나왔다.  
2)Number Of Sections: PE 파일은 code,data,rsrc로 각각의 섹션에 나뉘어서 저장되는데, 그 섹션의 크기를 나타낸다. 이 값은 반드시 0보다 커야하며, 정의된 섹션 개수와 실제 섹션이 다르면 에러가 발생한다.  
3)Size Of Optional Header: NT header 구조체의 마지막 멤버인 IMAGE_OPTIONAL_HEADER32(PE32) 구조체의 크기를 나타낸다.(64bit(PE 32+)인 경우 IMAGE_OPTIONAL_HEADER64)  
4)Charactieristics: 파일의 속성을 나타내는 값. (기억 해둘 것->0002h=실행파일이다,2000h=DLL 파일이다.)

![image](https://user-images.githubusercontent.com/65746019/116531310-01d65680-a91a-11eb-9281-048b4fcba81f.png)
상단 좌측부터: machine, number of sections, time date stamp, offset to symbol table, number of symbols, size of optional header, characteristics.  

## NT header- Optional Header (IMAGE_OPTIONAL_HEADER 32 기준으로 작성, 그림은 64임)  
1)Magic: 32bit 일 경우 10B, 64bit 경우는 20B이다.  
2)Address Of Entry Point: Entry point의 RVA 값을 가지고 있다. 프로그램 최초로 실행되는 코드의 시작 주소이다.(자주 사용할 것으로 보임)  
3)Image Base: 가상 메모리에서 PE 파일이 로딩되는 시작 주소. PE loader는 PE 파일을 실행하기 위해 프로세스를 디스크에서 메모리로 로딩한 뒤, EIP reg 값을 imageBase+ AddressOfEntry Point의 값으로 세팅한다. EXE,DLL 파일은 user memory 공간인 0-7FFF'FFFF'FFFF'FFFF을 사용하고 system file은 8000'0000'0000'0000공간부터 FFFF'FFFF'FFFF'FFFF까지 사용한다.  
4)Section Alignment, FileAlignment: 파일에서 섹션의 최소단위를 SectionAlignment에 메모리에서 섹션의 최소단위를 fileAlignment라 하고 각각의 섹션의 크기는 alignment 값의 배수가 되어야한다. 두개의 값이 같을수도 다를수도 있다.  
5)Size Of Image: PE 파일이 메모리에 로딩되었을 때, 가상 메모리에서 PE Image가 차지하는 크기.  
6)Size Of Header: PE 헤더의 전체 크기(file alignment의 배수). 시작부분에서 Size of Header offset 만큼 떨어진 위치에서 섹션 있음.  
7)Subsystem: 파일의 형태(sys,exe,dll) 구분가능함.(1: driver file, 2: GUI file(클릭=창 기반 app), 3: CUI file(명령어=콘솔 기반 app))  
8)NumverOfRvaAndSizes: optional header의 마지막 멤버 datadirectory 배열의 개수.  
9)Data directory: 어느 구조체의 배열(IMAGE_EXPORT_DIRECTORY, IMAGE_IMPORT_DIRECTORY...)로 각자 하나의 구조체를 담당한다.  

![image](https://user-images.githubusercontent.com/65746019/116543790-47e6e680-a929-11eb-8c87-ccc607004976.png)  

***

# Section Header  
PE 파일에는 여러개의 섹션이 존재하므로 각 섹션별로 특성, 접근 권한을 적어놓는 헤더가 바로 section header다.  
code- 실행, 읽기 권한  
data- 비실행, 읽기,쓰기 권한  
resource- 비실행, 읽기 권한  

## IMAGE_SECTION HEADER  
섹션 헤더는 각 섹션별로 이 구조체의 배열로 되어 있다.  
0)Name: .text,.data 같이 ascii 값이 들어갈수도 무엇이 들어가도 되며 참고용으로만 사용하자.  
1)VirtualSize: 메모리에서 섹션이 창지하는 크기  
2)VirtualAddress: 메모리에서 섹션의 시작 주소(RVA)  
3)SizeOfRawData: 파일에서 섹션이 차지하는 크기  
3과 4사이) offset 정보들..
4)PointerToRawData: 파일에서 섹션의 시작 위치  
5)characteristics: 섹션의 속성  

virtualaddress와 pointertorawdata는 optional header의 sectionalignment와 filealignment의 배수 배의 크기를 갖는다.  
또, virtualSize와 Sizeofrawdata는 일반적으로 다른 값을 가진다.(가상메모리상과 파일에서의 섹션의 크기는 다르다.)  

characteristics는 아래 그림의 or연산으로 조합되어 사용된다.  
![image](https://user-images.githubusercontent.com/65746019/116555382-46bcb600-a937-11eb-9510-9e6dfaf1abd9.png)  

![image](https://user-images.githubusercontent.com/65746019/116560495-66a2a880-a93c-11eb-87d3-70d240286e87.png)  
위의 그림에는 총 6개의 섹션이 존재한다.  

***

# RVA to RAW  
PE 파일이 메모리에 로딩 되었을 때 각 섹션에서 RVA와 파일 offset을 매핑하는 방법을 RVA to RAW라 한다.  
매핑 방법:  
1) RVA가 속한 섹션 찾음  
2) RAW= RVA- VA + PointerToRawData  

![image](https://user-images.githubusercontent.com/65746019/116562300-ff85f380-a93d-11eb-9470-46d7a9af2b4b.png)  

RVA가 5000이고 다음 그림에서 file offset?
위 상황에서 계산하면 text의 RVA가 1000이고 다음 섹션은 RVA가 24A00이니까 text 섹션 안이다.  
RAW= 5000(RVA)-1000(VA)+ 400(ptrtorawdata)=4400  

file offset 과 RVA가 같은 섹션이면 상관없는데 서로 다른 섹션에 잡히게 되면 정의할 수 없는 상태이다.  


# IAT  


