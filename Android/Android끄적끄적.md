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

## Parcelable vs Serializable
면접 단골 문제. 
 간단히 정리 해보자면, Serializable은 단순 Marker Inteface. 구현은 간단하지만, 데이터를 이동시키는데 리플렉션을 사용하기 때문에 많은 오브젝트들이 생기고 그에 따른 Garbage Collection을 발생시킴. 안드로이드 앱의 성능을 낮추고 베터리 성능을 낮출 수 있음.
 
 Parcelable은 프로그래머가 구현해주어야 할 코드들이 더 많음. 그 만큼 유지보수가 힘들 수 있지만, 속도가 훨씬 빠름. 그 이유는 전달하는데 있어서 IPC(Inter Process Communication) 매커니즘을 사용. 프로세스간 메모리 영역을 공유할 수 없기 떄문에, 커널 메모리를 통해 전달. Parcelable을 구현한 클래스는 추가 작업 없이 Process사이를 넘나 들 수 있음. 리플렉션 사용x.
 Parcel은 IPC 전용 데이터로 만들어진 클래스. Parcel은 '소포'라는 의미로 IPC시 이 소포를 전달해줌. 소포에 대한 모든 내용은 다 명시해 놨으니(프로그래머가 구현하는 그 보일러플레이트 코드들), IPC떄는 해당 프로세스로 전달만 하면 된다는 뜻임.
 또한, Parcelable을 구현한 클래스를 만들 때, 
 
    public static final Parcelable.Creator<SomeObject> CREATOR = new Parcelable.Creator<SomeObject>(){
        override fun createFromParcel(parcel: Parcel): SomeObject {
            val someObject = SomeObject()
            someObject.property1 = parcel.readInt()
            someObject.property2 = parcel.readString()
            ...
            return someObject
        }
    }

이러한 필드를 만들어 줘야 함(createFromParcel() -> unmarshalling과정). 여기서 구체적으로 <SomeObject>타입이 들어간 것을 볼 수 있음. 나중에 받는 쪽에서 intent.getParcelable("someObject") 할 때, 구체적인 타입을 받을 수 있음. 반면, Serializable을 통해서 구현하면, 받아와서 형 변환해야 함.
getParcelable()의 리턴 타입은 <T extends Parcelable>이고, getSerializable()의 리턴 타입은 Serializable임.
 마지막으로, unmarshalling이 있으면, marshalling과정이 있어야 되는데, writeToParcel()을 통해 구현. 이 때 중요한 것은, unmarshalling할 때, marshalling(override fun writeToParcel(parcel: Parcel, flags: Int) {...})과정에서 쓴 순서와 동일하게 작성해 줘야함.
