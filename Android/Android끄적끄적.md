# 안드로이드 끄적끄적


## 안드로이드 메모리 관리 방식
안드로이드는 액티비티, 서비스, 리시버, 프로바이더를 실행하기 위해 앱이 샐행되는 과정에서 프로세스를 생성. 실행 중인 모든 앱은 컴포넌트가 모두 종료 되어도
다음에 이 앱을 다시 실행할 가능성이 높기 때문에 프로세스를 바로 제거하지 않는다. 바로 종료하지 않는 이유는 앱을 실행하기 위해 프로세스를 생성하는 과정에서
딜레마가 발생하는데, 이 딜레마를 줄이기 위함이다. 따라서, 사용자에 의해 다시 앱이 실행되면 남아있던 프로세스가 존재하는 경우 바로 실행됩니다.
이 과정에서 쌓여있던 많은 프로세스로 인해 메모리가 부족해지는 경우 프로세스의 우선순위(사용빈도)에 따라 프로세스를 종료하여 메모리를 확보한다.


## Zygote란
zygote란 application을 빠르게 구동하기 위하여 미리 fork되어있는 process이다. system에서 exec() 호출을 통해 특정 application을 실행하고자 하기 전까지는 중립적인 상태를 유지하며 대기하고 있는 process이다. 공통적으로 필요한 libraries를 탑재한 상태로 반쯤 생성이 되어 application실행에 대비. 

구동속도 빨라짐 -> 실행되면 davik vm, system server실행. -> classes와 resources를 preloading한 상태로 application의 로직을 기다리는 중(없었다면 구동 시에 필요한 모든 부분을 찾아야되서 delay)

init process에 의해 구동! 되고 -> zygote는 -> 핵심이 되는 android system service및 application을 실행 -> System Server
