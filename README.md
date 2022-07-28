# MVVM with kotlin flow

## Creating android app using MVVM + Coroutines + Flow + Hilt

![](https://miro.medium.com/max/1400/1*jA2EBb6QyGgizm0reHqbKg.png)
In this tutorial, I will explain the MVVM architecture with kotlin coroutines and kotlin flow with hilt dependency injection. Also, I will explain the Room database with the flow in this tutorial.
![](https://miro.medium.com/max/1400/1*zgURZZJd4L9M_nEnyODXxg.png)
let’s begin with a short introduction about the features I will use in this tutorial.
## What is MVVM?
MVVM (Model-View-ViewModel) pattern helps to separate the business and presentation logic from the UI completely, and the business logic and UI can be separated for easier testing and maintenance. Let’s take a look at View, ViewModel, and Model.
![](https://miro.medium.com/max/1400/1*AW9NVFBAkhMOl3lLtIm5NQ.png)
## View
The view is responsible for the layout structure displayed on the screen. You can also execute UI logic.
## ViewModel
The ViewModel implements the data and commands connected to the View to notify the View of state changes via change notification events. Then, the View that receives the state change notification determines whether to apply the change.
## Model
Model is a non-visual class that has the data to use. Examples include DTO (Data Transfer Objects), POJO (Plain Old Java Objects), and entity objects. It is a commonly used service or repository that needs to access or cache data.

If you want to learn more about MVVM, check the below tutorials,
[MVVM with Kotlin Coroutines and Retrofit [Example] — Howtodoandroid](https://howtodoandroid.com/mvvm-kotlin-coroutines-retrofit/)
[MVVM With Retrofit and Recyclerview in Kotlin [Example] (howtodoandroid.com)](https://howtodoandroid.com/mvvm-retrofit-recyclerview-kotlin/)
## Basic of Kotlin Coroutines
A coroutine is a concurrency design pattern that you can use on Android to simplify code that executes asynchronously. Coroutines were added to Kotlin in version 1.3 and are based on established concepts from other languages.

The main advantages of using Coroutines
- They are light-weight
- Built-in cancellation support
- Lower chances for memory leaks
- Jetpack libraries provide coroutines support

## Introduction to Kotlin flow
- The Flow is a type that can emit multiple values sequentially, as opposed to suspending functions that return only a single value. For example, you can use a flow to receive live updates from a database.
- Flows are built on top of coroutines and can provide multiple values.
- A flow is conceptually a stream of data that can be computed asynchronously.
- The emitted values must be of the same type.

![](https://miro.medium.com/max/1400/0*f2RXL7uDL9Jn6v5q)
## Hilt dependency injection
The hilt is a dependency injection library for Android that reduces the boilerplate of doing manual dependency injection in your project. Hilt provides a standard way to use DI in your application by automatically providing containers for every Android class in your project and managing their lifecycles.

Also, check the below link to know more about the hilt,

[Dependency injection on Android with Hilt[Example] — Howtodoandroid](https://howtodoandroid.com/android-hilt-dependency-injection/)

Enough theory, let’s start with creating an android application using MVVM, kotlin coroutines, and flow.
### MVVM + Coroutines + Flow + Hilt Example
The first step is to add dependencies for the components, like ViewModel, Coroutines, Hilt, Flow, Room, Retrofit, etc.
```
 //coroutines
    implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.5.2'
//hilt
    implementation "com.google.dagger:hilt-android:2.38.1"
    kapt "com.google.dagger:hilt-compiler:2.38.1"
// retrofit
    implementation 'com.squareup.retrofit2:retrofit:2.9.0'
    implementation "com.squareup.okhttp3:okhttp:4.7.2"
    implementation "com.squareup.okhttp3:logging-interceptor:4.7.2"
    implementation 'com.squareup.retrofit2:converter-gson:2.9.0'
//ktx
    implementation "androidx.activity:activity-ktx:1.4.0"
    implementation "androidx.fragment:fragment-ktx:1.4.1"
    implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:2.4.1"
    implementation "androidx.lifecycle:lifecycle-runtime-ktx:2.4.1"
    // glide
    implementation 'com.github.bumptech.glide:glide:4.13.0'
    annotationProcessor 'com.github.bumptech.glide:compiler:4.13.0'
```
### Hilt Dependency injection setup
Set the application as HiltAndroidApp.
 ```
 @HiltAndroidApp
class MyApplication: Application() {
override fun onCreate() {
        super.onCreate()
    }
}
```
Also, don’t forget to add the application class and internet permission to the AndroidManifest.xml file.
```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    package="com.example.mvvmwithretrofitandflow">
<uses-permission android:name="android.permission.INTERNET"/>
<application
        android:name=".MyApplication"
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:usesCleartextTraffic="true"
        android:theme="@style/Theme.MVVMWithRetrofitAndFlow"
        tools:targetApi="31">
        <activity
            android:name=".ui.MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
<category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
</manifest>
```
create the network module for retrofit setup.
#### NetworkModule.kt

```
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {
    
    @Singleton
    @Provides
    fun provideOkHttp() : OkHttpClient{
        return OkHttpClient.Builder()
            .build()
    }
@Singleton
    @Provides
    @Named("loggingInterceptor")
    fun provideLoggingInterceptor(): HttpLoggingInterceptor {
        return HttpLoggingInterceptor().apply {
            this.level = HttpLoggingInterceptor.Level.BODY
        }
    }
@Provides
    fun provideRetrofit(okHttpClient: OkHttpClient): Retrofit {
        return Retrofit.Builder()
            .baseUrl("https://imdb-api.com/en/")
            .addConverterFactory(GsonConverterFactory.create())
            .client(okHttpClient)
            .build()
    }
@Provides
    fun provideApiClient(retrofit: Retrofit): ApiService {
        return retrofit.create(ApiService::class.java)
    }
    
}
```
for the API calls, I am using https://imdb-api.com/en/. To use this API need an account with IMDb-api.com.

and the Response Model, Model.kt
```
@Entity
data class Movie(
    @PrimaryKey
    val id: String,
    val title: String,
    val year: String,
    val image: String,
    val imDbRating: String
)
data class MovieResponse(val items: List<Movie>, val errorMessage: String)
```

### Repository setup with flow
On the repository, I am calling the API service and returning the response as flow to ViewModel. Also, I am using NetworkResult sealed class to emit different statuses of the network call.

#### NetworkResult.kt
```
sealed class NetworkResult<T> {
    data class Loading<T>(val isLoading: Boolean) : NetworkResult<T>()
    data class Success<T>(val data: T) : NetworkResult<T>()
    data class Failure<T>(val errorMessage: String) : NetworkResult<T>()
}
```
#### MainRepository.kt
```
class MainRepository @Inject constructor(private val apiService: ApiService) {
suspend fun getPopularMovies()  = flow {
        emit(NetworkResult.Loading(true))
        val response = apiService.getMostPopularMovies()
       emit(NetworkResult.Success(response.items))
    }.catch { e ->
        emit(NetworkResult.Failure(e.message ?: "Unknown Error"))
    }
}
```
### ViewModel setup with Livedata
In the ViewModel collect the flow and set the response to livedata for UI changes.
```
@HiltViewModel
class MainViewModel @Inject constructor(
    private val mainRepository: MainRepository
) : ViewModel() {
private var _movieResponse = MutableLiveData<NetworkResult<List<Movie>>>()
    val movieResponse: LiveData<NetworkResult<List<Movie>>> = _movieResponse
init {
        fetchAllMovies()
    }
private fun fetchAllMovies() {
        viewModelScope.launch {
            mainRepository.getPopularMovies().collect {
                _movieResponse.postValue(it)
            }
        }
    }
}
```
### Observe the changes in UI
finally, I have observed the changes from ViewModel to UI class.
```
@AndroidEntryPoint
class MainActivity : AppCompatActivity() {
private lateinit var binding: ActivityMainBinding
private val mainViewModel: MainViewModel by viewModels()
    @Inject
    lateinit var movieAdapter: MovieAdapter
override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
binding.rvMovies.adapter = movieAdapter
movieAdapter.setItemClick(object : ClickInterface<Movie> {
            override fun onClick(data: Movie) {
                Toast.makeText(this@MainActivity, data.title, Toast.LENGTH_SHORT).show()
            }
        })
mainViewModel.movieResponse.observe(this) {
            when(it) {
                is NetworkResult.Loading -> {
                    binding.progressbar.isVisible = it.isLoading
                }
is NetworkResult.Failure -> {
                    Toast.makeText(this, it.errorMessage, Toast.LENGTH_SHORT).show()
                    binding.progressbar.isVisible = false
                }
is  NetworkResult.Success -> {
                    movieAdapter.updateMovies(it.data)
                    binding.progressbar.isVisible = false
                }
            }
        }
}
}
```
that’s it. we have done with the coding let’s run the code and check it. Now you can able to see the list of movies in recyclerview.

![](https://miro.medium.com/max/1400/1*Q_0v0dFypqQqSiuCh7SDdg.png#x40%)

Source: https://medium.com/android-beginners/creating-android-app-using-mvvm-coroutines-flow-hilt-a8acf7f57630

