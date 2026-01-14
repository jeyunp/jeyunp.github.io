---

title: PintOS 설치
date: 2022-04-01 13:30:00 +0900
categories: [Study, PintOS]
tags: [PintOS]
comments: true

---


# PintOS 설치

최근 운영체제 관련 내용 학습을 위해 PintOS 프로젝트를 시작했다.

PintOS 학습을 할 수 있도록 준비하는 과정이 생각보다 까다로웠기 때문에 당황했다. PintOS를 처음에 빌드하는 과정에서 인터넷에 나와 있는 상당수의 해결책을 따라했음에도 다른 부분에서 실수를 한다던가 하는 등의 이유로 여러 번 오류가 발생했으므로 글을 남겨보도록 하겠다.

pintOS를 설치한 환경은 conoHa 클라우드 VPS에 설치된 우분투 20.04 LTS이다. pintOS를 설치한 현재까지는 그냥 로컬에 리눅스 설치해서 작업하는 것과 비교해 따로 고려해줄 점은 없는 듯 하다. 

## 1.qemu 설치
매우 간단하다.
```
$sudo apt-get install qemu
```
하면 된다.

그런데, 이렇게만 하고 나중에 실행을 시키려고 보니 프로그램에서 qemu-system-x86_64(혹은 i386)가 뭔지를 모르겠다고 하는게 아닌가? 보통 인터넷에 돌아다니는 pintOS 설치 관련 자료들에서는 install qemu로 충분했는데, 만약 위와 같은 에러가 발생한다면 다음과 같이 해주자
```
$sudo apt-get install qemu-system
```
위의 명령어는 qemu-system 패키지를 설치시켜주는데, 해당 패키지 안에는 qemu-system-x86_64와 qemu-system-i386이 포함되어 있다.

## 2.PintOS 소스코드 다운로드
다음과 같은 명령어를 통해 PintOS 소스코드를 다운로드받을 수 있다.
```
$git clone git://pintos-os.org/pintos-anon
```

## 3.PintOS 빌드
먼저 몇 가지 변수를 수정해주어야 한다. 에디터는 편의상 nano를 사용하였다.

먼저 pintos-gdb 파일 내의 GDBMACROS를 수정해야 한다.
```
$cd pintos-anon/src/utils
$nano pintos-gdb
```
GDBMACROS를 pintos directory의 full path를 가리키도록 수정한다. 즉,
```
GDBMACROS=/home/...(pintos-anon까지의 경로)/pintos-anon/src/misc/gdb-macros
```
로 수정해 준다.

그리고 나서 동일 디렉터리의 Makefile 파일을 텍스트 에디터로 열어 LOADLIBES라는 단어를 LDLIBS로 수정한다.

다음으로, make를 해준다. /src/utils 디렉터리에서 make를 실행해야 한다.
```
$ make
```
아마도 이 단계에서 stropts.h와 관련된 에러가 발생할 것이다. 찾아보니 아주 옛날에 쓰던 라이브러리고 현재에는 사용하지 않는 라이브러리라고 한다. 인터넷에 소스코드를 수정하지 않고 문제를 해결할 수 있는 몇 가지 해결책이 나와 있기는 한데 대부분 Fedora 사용자를 위한 해결책이라 우분투 환경에서는 사용하기 어려운 해결책인 듯 하다. 그래서 (사실 이렇게 해도 되는지는 아직도 잘 모르겠다.)문제가 되는 해당 코드들을 주석 처리하는 방법으로 해결했다.

즉. squish-pty.c와 squish-unix.c의 #include <stropts.h> 구문과 squish-pty.c의 line 288-293을 주석 처리 해준다. 그리고 나서 make를 실행하면 정상적으로 처리가 진행될 것이다.


이제는 threads 디렉터리로 이동하여 작업한다.
```
$cd ../threads
```
Make.vars 파일을 열어 7번 라인의 bochs를 qemu로 수정한다.
```
$nano Make.vars
```
```
SIMULATOR = --qemu
```
그리고 나서 여기서도 make를 해주자.
```
$make
```

그러면 threads 디렉터리 내에 build 디렉터리가 생겨나고, 안에 kernel.bin, loader.bin 파일이 있는 것을 확인할 수 있다. 잠시 후에 해당 파일들의 full path를 이용하니 잘 기억해 놓아야 한다.


이제 거의 막바지 단계이다. 다시 utils 디렉터리로 이동하여 pintos파일의 내용을 수정할 것이다.
```
$cd ../utils
$nano pintos
```
line 103쯤의 bochs를 qemu로 바꿔 써준다.
```
$sim = "qemu" if !defined $sim;
```
line 257쯤의 kernal.bin을 kernal.bin의 full path로 수정해 준다.
```
my $name = find_file('/home/...(중간 경로)/pintos-anon/src/threads/build/kernel.bin')
```
line 621쯤의 qemu를 qemu-system-x86_64로 수정한다.(사실 웹 상 자료들을 보면 이렇게 하는 자료가 있고 안 하는 자료가 있는데... 아마 수정을 안 하는 자료들은 qemu랑 qemu-system-x86_64간에 링크를 만들어준 것 같다.이런 경우에 아마도 위의 qemu-system을 인스톨하는 과정이 필요가 없어지겠지...)
```
my (@cmd) = ('qemu-system-x86_64');
```
마지막으로 Pintos.pm파일의 362번 라인 쯤의 loader.bin을 loader.bin의 full path로 수정한다.
```
$nano Pintos.pm
```
```
$name = find_file ("/home/...(중간 경로)/pintos-anon/src/threads/build/loader.bin") if !defined $name;
```
이때 주의사항으로, kernal.bin이나 loader.bin 경로를 입력 시 perl이 처리 가능한 문자를 이용하여야 한다. 즉, 예를 들어 다음과 같은 방식으로 경로를 입력하면 안 된다.
```
/$HOME/...(중간 경로)/pintos-anon/src/threads/build/loader.bin
```

## 4.PintOS 동작 확인
/src/utils 디렉터리에서 실행해본다.
```
$./pintos run alarm-multiple
```

## 5.PATH 설정
PintOS를 항상 /src/utils 디렉터리에서 실행하기는 귀찮고 불편하다. 따라서, 다른 디렉터리에서도 실행 가능하도록 PATH를 설정해주자.
```
$nano ~/.bashrc
```
에디터로 마지막 줄에 다음 문장을 추가한다.
```
export PATH=/home/...(중간 경로)/pintos/src/utils:$PATH
```
그리고 terminal을 reload 한다.
```
$source ~/.bashrc
```
이제 어느 디렉터리에서나 다음과 같은 명령어로 pintOS를 실행 가능하다.(예시는 alarm-multiple)
```
$pintos run alarm-multiple
```

## 6.참고자료
https://stackoverflow.com/questions/60696354/cloning-pintos-with-ubuntu
https://github.com/kumardeepakr3/PINTOS-Ubuntu
https://scefuvh.github.io/install_pintos_on_archlinux/
http://absolutelyrandomshit.blogspot.com/2017/09/errors-while-installing-pintos.html
https://askubuntu.com/questions/138140/how-do-i-install-qemu


