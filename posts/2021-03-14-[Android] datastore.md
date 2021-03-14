# DataStore


DataStore는 Jetpack에 나온 새로운 데이터 스토리지 솔루션이다.
Kotlin 코루틴 및 Flow를 사용하여 비동기적으로 접근한다.

DataStore 사용법은 2가지가 있다.
1. Preferences DataStore - Shared Preferences과 비슷하게 Key-Value형태의 read, write
2. Proto DataStore - protocol buffers를 사용한 스키마 정의. 이를 이용한 typed object의 저장.


<br/>


## SharedPreferences와 차이


- Lack of a fully asynchronous API
 - Lack of main thread safety
 - No type safety

DataStore는 이러한 문제점들을 해결해서 '비동기적인 API, main thread safety'를 기본으로 한다. 모든 작업은 내부적으로 Dispatchers.IO로 이동한다.


<br/>


## Preferences DataStore


#### 디펜던시 추가

    implementation "androidx.datastore:datastore-preferences:1.0.0-alpha01"
    
#### Create

    val dataStore: DataStore<Preferences> = context.createDataStore(
        name = "settings"
    )
    
#### Read

    val MY_COUNTER = preferencesKey<Int>("my_counter")
    val myCounterFlow: Flow<Int> = dataStore.data
         .map { currentPreferences ->
            currentPreferences[MY_COUNTER] ?: 0   
         }
         
#### Write

    suspend fun incrementCounter() {
        dataStore.edit { settings ->
            val currentCounterValue = settings[MY_COUNTER] ?: 0
            settings[MY_COUNTER] = currentCounterValue + 1
        }
    }


<br/>


## Proto DataStore

type safety하다.


#### 디펜던시 추가

    implementation  "androidx.datastore:datastore-core:1.0.0-alpha01"


#### Proto Objects 정의

Proto DataStore를 사용할 때는, app/src/main/proto 디렉토리에 proto file 스키마를 정의한다.

    syntax = "proto3";      // proto3 syntax를 쓰겠다

    option java_package = "<your package name here>";     // 컴파일러에게 어디 위치에 generated 파일들을 위치시키는지 알린다
    option java_multiple_files = true;      // 최상위 'message'에 대해 별도 파일을 생성해라

    message Settings {      // 'message' 키워드를 통해 구조를 정의. 멤버, 필드 등..
      int my_counter = 1;
    }
    
정의가 끝나고 rebuild하면, 파일들이 generate됨.

![image](https://user-images.githubusercontent.com/19990905/111068966-3929a900-850e-11eb-9d93-3953398741b7.png)

(위치만 참조)

    
#### Serializer 생성

Serializer interface를 구현해서 DataStore에게 데이터를 어떻게 data type을 읽고 쓰는지 알려줘야함.

    object SettingsSerializer : Serializer<Settings> {
        override fun readFrom(input: InputStream): Settings {
            try {
                return Settings.parseFrom(input)
            } catch (exception: InvalidProtocolBufferException) {
                throw CorruptionException("Cannot read proto.", exception)
            }
        }

        override fun writeTo(t: Settings, output: OutputStream) = t.writeTo(output)
    }

    // with Proto DataStore
    val settingsDataStore: DataStore<Settings> = context.createDataStore(
        fileName = "settings.pb",
        serializer = SettingsSerializer
    )

위 Settings는 proto file의 message를 통해 정의한 Settings.


#### Read

    val myCounterFlow: Flow<Int> = settingsDataStore.data
        .map { settings ->
            // The myCounter property is generated for you from your proto schema!
            settings.myCounter      // proto file에 my_counter
        }

#### Write

    suspend fun incrementCounter() {
        settingsDataStore.updateData { currentSettings ->
            // We can safely increment our counter without losing data due to races!
            currentSettings.toBuilder()
                .setMyCounter(currentSettings.myCounter + 1)
                .build()
        }
    }
    
    
</br>
    

## SharedPreferences에서 DataStore로 Migration


이것도 2가지로 나눠서

#### with Preferences DataStore

    val dataStore: DataStore<Preferences> = context.createDataStore(
        name = "settings",
        migrations = listOf(SharedPreferencesMigration(context, "settings_preferences"))
    )


#### with Proto DataStore

    val settingsDataStore: DataStore<Settings> = context.createDataStore(
        produceFile = { File(context.filesDir, "settings.preferences_pb") },
        serializer = SettingsSerializer,
        migrations = listOf(
            SharedPreferencesMigration(
                context,
                "settings_preferences"            
            ) { sharedPrefs: SharedPreferencesView, currentData: UserPreferences ->
                // Map your sharedPrefs to your type here
              }
        )
    )


<br/>


마지막으로

## SharedPreferences VS DataStore

비교표

![Screen Shot 2020-08-31 at 11 25 43 AM](https://user-images.githubusercontent.com/19990905/111069375-27e19c00-8510-11eb-9374-d0c351c4b3aa.png)




참조 : 
https://android-developers.googleblog.com/2020/09/prefer-storing-data-with-jetpack.html
https://www.raywenderlich.com/18348259-datastore-tutorial-for-android-getting-started
https://developer.android.com/topic/libraries/architecture/datastore?gclsrc=aw.ds&&gclid=CjwKCAiAhbeCBhBcEiwAkv2cY1e7sE_iS34RfVcJoE90T-vtOtYdsn93uCNvZPLxsd22-iLCsQ_0ExoC10sQAvD_BwE
