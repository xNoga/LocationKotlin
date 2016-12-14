# LocationKotlin

This readme will guide you through the use of location in Kotlin. If you need to know how to use Location in Java there are plenty of guides and documentation on the web. 

The whole MainActivity file have been added at the bottom of this readme but we will go through every step regarding the use of location in Kotlin. 

First we should add the Google Play Service Location in our dependencies in order to use one of Google's many API's to help us access and use a bunch of different Android features. You can either compile all of Google's Play Services or specify which feature you want. We have chosen to import location only: 
```Kotlin
dependencies {
    compile 'com.google.android.gms:play-services-location:10.0.1'
}
```

Now we need to initialize the GoogleApiClient in MainActivity:

```Kotlin
lateinit var mLastLocation : Location // this is the property we can save the location in
lateinit var mGoogleApiClient : GoogleApiClient
// var mGoogleApiClient : GoogleApiClient? = null
```
In order to prevent us having to constantly check if mGoogleApiClient is null before use, we can use the property modifier <b>lateinit</b>.
We can use lateinit in situations where we need to instansiate an object that we need to use later in our code but at runtime the object should not be null. Lateinit will give mGoogleApiClient a non-null value but we cannot use mGoogleClient before we instantiate it ourself. If we try use it before instantiation we will get an exception. 

Next we need to instantiate mGoogleApiClient inside the onCreate() method:
```Kotlin
// Create an instance of GoogleAPIClient.
        
mGoogleApiClient = GoogleApiClient.Builder(this)
        .addConnectionCallbacks(this)
        .addOnConnectionFailedListener(this)
        .addApi(LocationServices.API)
        .build()
        
onStart()
```

Right after we call the onStart() metod. That method need to be overwritten and inside onStart() we add the following:
```Kotlin
override fun onStart() {
        mGoogleApiClient?.connect()
        super.onStart()
    }
```
This is where we connect the mGoogleApiClient and it will connect and try to get a location.

The addConnectionCallbacks and addOnConnectionFailed are callbacks that will be called if we connect succesfully or not. You can specify what should happen in these functions which you can see at the bottom of the readme. In this example the onConnected function should look like this: 
```Kotlin
override fun onConnected(p0: Bundle?) {
            mLastLocation = LocationServices.FusedLocationApi.getLastLocation(
                    mGoogleApiClient)
            if (mLastLocation != null) {
                NewPostActivity.myLocation = mLastLocation as Location
            } else {
                ActivityCompat.requestPermissions(this@MainActivity,
                        arrayOf(Manifest.permission.ACCESS_COARSE_LOCATION),
                        1)
            }

    }
```
Here we still need to check if mLastLocation is null because the user could have location services turned off on their device. If that is the case the location will be null. 
This is maybe not the right solution. We originally set the locations to null an therefore we had to check if the location was null first. Now when we know the locations are never null (because of lateinit) we could instead check if the user had given us permission to use location first, and then set the location. This step is not the correct way if you use lateinit. 
Now you have the users location in the property mLastLocation. In the example above we use this to set the value in a companion object in another class. 



```Kotlin
import com.google.android.gms.common.ConnectionResult
import com.google.android.gms.common.api.GoogleApiClient
import com.google.android.gms.location.LocationServices

class MainActivity : AppCompatActivity(), GoogleApiClient.OnConnectionFailedListener,
        GoogleApiClient.ConnectionCallbacks {
    lateinit var mGoogleApiClient : GoogleApiClient
    var mLastLocation : Location? = null

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // Create an instance of GoogleAPIClient.
        if (mGoogleApiClient == null) {
            mGoogleApiClient = GoogleApiClient.Builder(this)
                    .addConnectionCallbacks(this)
                    .addOnConnectionFailedListener(this)
                    .addApi(LocationServices.API)
                    .build()
        }
        onStart()
        var p1 = P_User(1, "Magnoob", "Lolsen")
        
    }
    
    override fun onStart() {
        mGoogleApiClient.connect()
        super.onStart()
    }

    override fun onStop() {
        mGoogleApiClient?.disconnect()
        super.onStop()
    }
    override fun onConnectionSuspended(p0: Int) {
        throw UnsupportedOperationException("not implemented") //To change body of created functions use File | Settings | File     Templates.
    }

    override fun onConnected(p0: Bundle?) {
            mLastLocation = LocationServices.FusedLocationApi.getLastLocation(
                    mGoogleApiClient)
            if (mLastLocation != null) {
                NewPostActivity.myLocation = mLastLocation as Location
            } else {
                ActivityCompat.requestPermissions(this@MainActivity,
                        arrayOf(Manifest.permission.ACCESS_COARSE_LOCATION),
                        1)
            }

    }
    
    override fun onRequestPermissionsResult(requestCode: Int,
                                            permissions: Array<String>, grantResults: IntArray) {
        when (requestCode) {
            1 -> {
                if (grantResults.size > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                    mLastLocation = LocationServices.FusedLocationApi.getLastLocation(mGoogleApiClient)
                    NewPostActivity.myLocation = mLastLocation as Location
                } else {
                    Toast.makeText(this@MainActivity, "Permission denied to get your location", Toast.LENGTH_SHORT).show()
                }
                return
            }
        }
    }
    
    override fun onConnectionFailed(p0: ConnectionResult) {
        toast("connection failed")
    }


```
