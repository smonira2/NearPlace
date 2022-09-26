# NearPlace
Maps Activity.Java
package com.app.besmartybd.nearplace;
import android.Manifest;
import android.content.pm.PackageManager;
import android.location.Location;
import android.os.Build;
import android.support.v4.app.ActivityCompat;
import android.support.v4.app.FragmentActivity;
import android.os.Bundle;
import android.support.v4.content.ContextCompat;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Toast;
import com.google.android.gms.common.ConnectionResult;
import com.google.android.gms.common.GoogleApiAvailability;
import com.google.android.gms.common.api.GoogleApiClient;
import com.google.android.gms.location.LocationListener;
import com.google.android.gms.location.LocationRequest;
import com.google.android.gms.location.LocationServices;
import com.google.android.gms.maps.CameraUpdateFactory;
import com.google.android.gms.maps.GoogleMap;
import com.google.android.gms.maps.OnMapReadyCallback;
import com.google.android.gms.maps.SupportMapFragment;
import com.google.android.gms.maps.model.BitmapDescriptorFactory;
import com.google.android.gms.maps.model.LatLng;
import com.google.android.gms.maps.model.Marker;
import com.google.android.gms.maps.model.MarkerOptions;
import java.util.HashMap;
import java.util.List;
public class MapsActivity extends FragmentActivity implements 
OnMapReadyCallback,
 GoogleApiClient.ConnectionCallbacks,
 GoogleApiClient.OnConnectionFailedListener,
 LocationListener {
 private GoogleMap mMap;
 double latitude;
 double longitude;
 private int PROXIMITY_RADIUS = 10000;
 GoogleApiClient mGoogleApiClient;
 Location mLastLocation;
 Marker mCurrLocationMarker;
 LocationRequest mLocationRequest;
 @Override
 protected void onCreate(Bundle savedInstanceState) {
 super.onCreate(savedInstanceState);
 setContentView(R.layout.activity_maps);
 if (android.os.Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
 checkLocationPermission();
 }
 //Check if Google Play Services Available or not
 if (!CheckGooglePlayServices()) {
 Log.d("onCreate", "Finishing test case since Google Play 
Services are not available");
 finish();
 }
 else {
 Log.d("onCreate","Google Play Services available.");
 }
 // Obtain the SupportMapFragment and get notified when the map is 
ready to be used.
 SupportMapFragment mapFragment = (SupportMapFragment) 
getSupportFragmentManager()
 .findFragmentById(R.id.map);
 mapFragment.getMapAsync(this);
 }
 private boolean CheckGooglePlayServices() {
 GoogleApiAvailability googleAPI = 
GoogleApiAvailability.getInstance();
 int result = googleAPI.isGooglePlayServicesAvailable(this);
 if(result != ConnectionResult.SUCCESS) {
 if(googleAPI.isUserResolvableError(result)) {
 googleAPI.getErrorDialog(this, result,
 0).show();
 }
 return false;
 }
 return true;
 }
 /**
 * Manipulates the map once available.
 * This callback is triggered when the map is ready to be used.
 * This is where we can add markers or lines, add listeners or move 
the camera. In this case,
 * we just add a marker near Sydney, Australia.
 * If Google Play services is not installed on the device, the user 
will be prompted to install
 * it inside the SupportMapFragment. This method will only be 
triggered once the user has
 * installed Google Play services and returned to the app.
 */
 @Override
 public void onMapReady(GoogleMap googleMap) {
 mMap = googleMap;
 mMap.setMapType(GoogleMap.MAP_TYPE_NORMAL);
 mMap.setMinZoomPreference(15);
 //Initialize Google Play Services
 if (android.os.Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
 if (ContextCompat.checkSelfPermission(this,
 Manifest.permission.ACCESS_FINE_LOCATION)
 == PackageManager.PERMISSION_GRANTED) {
 buildGoogleApiClient();
mMap.setMyLocationEnabled(true);
 }
 }
 else {
 buildGoogleApiClient();
 mMap.setMyLocationEnabled(true);
 }
 Button btnRestaurant = (Button) findViewById(R.id.btnRestaurant);
 btnRestaurant.setOnClickListener(new View.OnClickListener() {
 String Restaurant = "restaurant";
 @Override
 public void onClick(View v) {
 Log.d("onClick", "Button is Clicked");
 mMap.clear();
 String url = getUrl(latitude, longitude, Restaurant);
 Object[] DataTransfer = new Object[2];
 DataTransfer[0] = mMap;
 DataTransfer[1] = url;
Log.d("onClick", url);
 GetNearbyPlacesData getNearbyPlacesData = new 
GetNearbyPlacesData();
 getNearbyPlacesData.execute(DataTransfer);
Toast.makeText(MapsActivity.this,"Nearby Restaurants", 
Toast.LENGTH_LONG).show();
 }
 });
 Button btnHospital = (Button) findViewById(R.id.btnHospital);
 btnHospital.setOnClickListener(new View.OnClickListener() {
 String Hospital = "hospital";
 @Override
 public void onClick(View v) {
 Log.d("onClick", "Button is Clicked");
 mMap.clear();
 String url = getUrl(latitude, longitude, Hospital);
 Object[] DataTransfer = new Object[2];
 DataTransfer[0] = mMap;
 DataTransfer[1] = url;
Log.d("onClick", url);
 GetNearbyPlacesData getNearbyPlacesData = new 
GetNearbyPlacesData();
 getNearbyPlacesData.execute(DataTransfer);
 Toast.makeText(MapsActivity.this,"Nearby Hospitals", 
Toast.LENGTH_LONG).show();
 }
 });
 Button btnSchool = (Button) findViewById(R.id.btnSchool);
 btnSchool.setOnClickListener(new View.OnClickListener() {
 String School = "school";
 @Override
 public void onClick(View v) {
 Log.d("onClick", "Button is Clicked");
 mMap.clear();
 if (mCurrLocationMarker != null) {
 mCurrLocationMarker.remove();
 }
String url = getUrl(latitude, longitude, School);
 Object[] DataTransfer = new Object[2];
 DataTransfer[0] = mMap;
 DataTransfer[1] = url;
Log.d("onClick", url);
 GetNearbyPlacesData getNearbyPlacesData = new 
GetNearbyPlacesData();
 getNearbyPlacesData.execute(DataTransfer);
Toast.makeText(MapsActivity.this,"Nearby Schools", 
Toast.LENGTH_LONG).show();
 }
 });
 }
 protected synchronized void buildGoogleApiClient() {
 mGoogleApiClient = new GoogleApiClient.Builder(this)
 .addConnectionCallbacks(this)
 .addOnConnectionFailedListener(this)
 .addApi(LocationServices.API)
 .build();
 mGoogleApiClient.connect();
 }
 @Override
 public void onConnected(Bundle bundle) {
 mLocationRequest = new LocationRequest();
 mLocationRequest.setInterval(1000);
 mLocationRequest.setFastestInterval(1000);
 
mLocationRequest.setPriority(LocationRequest.PRIORITY_BALANCED_POWER_ACCUR
ACY);
 if (ContextCompat.checkSelfPermission(this,
 Manifest.permission.ACCESS_FINE_LOCATION)
 == PackageManager.PERMISSION_GRANTED) {
 
LocationServices.FusedLocationApi.requestLocationUpdates(mGoogleApiClient, 
mLocationRequest, this);
 }
 }
 private String getUrl(double latitude, double longitude, String 
nearbyPlace) {
 StringBuilder googlePlacesUrl = new 
StringBuilder("https://maps.googleapis.com/maps/api/place/nearbysearch/jso
n?");
 googlePlacesUrl.append("location=" + latitude + "," + longitude);
 googlePlacesUrl.append("&radius=" + PROXIMITY_RADIUS);
 googlePlacesUrl.append("&type=" + nearbyPlace);
 googlePlacesUrl.append("&sensor=true");
 googlePlacesUrl.append("&key=" + 
"AIzaSyATuUiZUkEc_UgHuqsBJa1oqaODI-3mLs0");
 Log.d("getUrl", googlePlacesUrl.toString());
 return (googlePlacesUrl.toString());
 }
 @Override
 public void onConnectionSuspended(int i) {
 }
 @Override
 public void onLocationChanged(Location location) {
 Log.d("onLocationChanged", "entered");
 mLastLocation = location;
 if (mCurrLocationMarker != null) {
 mCurrLocationMarker.remove();
 }
 //Place current location marker
 latitude = location.getLatitude();
 longitude = location.getLongitude();
 LatLng latLng = new LatLng(location.getLatitude(), 
location.getLongitude());
 MarkerOptions markerOptions = new MarkerOptions();
 markerOptions.position(latLng);
 markerOptions.title("Current Position");
 
markerOptions.icon(BitmapDescriptorFactory.defaultMarker(BitmapDescriptorF
actory.HUE_MAGENTA));
 mCurrLocationMarker = mMap.addMarker(markerOptions);
 //move map camera
 mMap.moveCamera(CameraUpdateFactory.newLatLng(latLng));
 mMap.animateCamera(CameraUpdateFactory.zoomTo(11));
 Toast.makeText(MapsActivity.this,"Your Current Location", 
Toast.LENGTH_LONG).show();
 Log.d("onLocationChanged", String.format("latitude:%.3f 
longitude:%.3f",latitude,longitude));
 //stop location updates
 if (mGoogleApiClient != null) {
 
LocationServices.FusedLocationApi.removeLocationUpdates(mGoogleApiClient, 
this);
 Log.d("onLocationChanged", "Removing Location Updates");
 }
 Log.d("onLocationChanged", "Exit");
 }
 @Override
 public void onConnectionFailed(ConnectionResult connectionResult) {
 }
 public static final int MY_PERMISSIONS_REQUEST_LOCATION = 99;
 public boolean checkLocationPermission(){
 if (ContextCompat.checkSelfPermission(this,
 Manifest.permission.ACCESS_FINE_LOCATION)
 != PackageManager.PERMISSION_GRANTED) {
 // Asking user if explanation is needed
 if (ActivityCompat.shouldShowRequestPermissionRationale(this,
 Manifest.permission.ACCESS_FINE_LOCATION)) {
 // Show an explanation to the user *asynchronously* --
don't block
 // this thread waiting for the user's response! After the 
user
 // sees the explanation, try again to request the 
permission.
 //Prompt the user once explanation has been shown
ActivityCompat.requestPermissions(this,
 new 
String[]{Manifest.permission.ACCESS_FINE_LOCATION},
 MY_PERMISSIONS_REQUEST_LOCATION);
 } else {
 // No explanation needed, we can request the permission.
ActivityCompat.requestPermissions(this,
 new 
String[]{Manifest.permission.ACCESS_FINE_LOCATION},
 MY_PERMISSIONS_REQUEST_LOCATION);
 }
 return false;
 } else {
 return true;
 }
 }
 @Override
 public void onRequestPermissionsResult(int requestCode,
 String permissions[], int[] 
grantResults) {
 switch (requestCode) {
 case MY_PERMISSIONS_REQUEST_LOCATION: {
 // If request is cancelled, the result arrays are empty.
 if (grantResults.length > 0
 && grantResults[0] == 
PackageManager.PERMISSION_GRANTED) {
 // permission was granted. Do the
// contacts-related task you need to do.
if (ContextCompat.checkSelfPermission(this,
 Manifest.permission.ACCESS_FINE_LOCATION)
 == PackageManager.PERMISSION_GRANTED) {
 if (mGoogleApiClient == null) {
 buildGoogleApiClient();
 }
 mMap.setMyLocationEnabled(true);
 }
 } else {
 // Permission denied, Disable the functionality that 
depends on this permission.
 Toast.makeText(this, "permission denied", 
Toast.LENGTH_LONG).show();
 }
return;
 }
 // other 'case' lines to check for other permissions this app 
might request.
 // You can add here other case statements according to your 
requirement.
 }
 }
}
GetNearbyPlacesData. Java
package com.app.besmartybd.nearplace;
import android.os.AsyncTask;
import android.util.Log;
import com.google.android.gms.maps.CameraUpdateFactory;
import com.google.android.gms.maps.GoogleMap;
import com.google.android.gms.maps.model.BitmapDescriptorFactory;
import com.google.android.gms.maps.model.LatLng;
import com.google.android.gms.maps.model.MarkerOptions;
import org.json.JSONObject;
import java.util.HashMap;
import java.util.List;
public class GetNearbyPlacesData extends AsyncTask<Object, String, String> 
{
 String googlePlacesData;
 GoogleMap mMap;
 String url;
 @Override
 protected String doInBackground(Object... params) {
 try {
 Log.d("GetNearbyPlacesData", "doInBackground entered");
 mMap = (GoogleMap) params[0];
 url = (String) params[1];
 DownloadUrl downloadUrl = new DownloadUrl();
 googlePlacesData = downloadUrl.readUrl(url);
 Log.d("GooglePlacesReadTask", "doInBackground Exit");
 } catch (Exception e) {
 Log.d("GooglePlacesReadTask", e.toString());
 }
 return googlePlacesData;
 }
 @Override
 protected void onPostExecute(String result) {
 Log.d("GooglePlacesReadTask", "onPostExecute Entered");
 List<HashMap<String, String>> nearbyPlacesList = null;
 DataParser dataParser = new DataParser();
 nearbyPlacesList = dataParser.parse(result);
 ShowNearbyPlaces(nearbyPlacesList);
 Log.d("GooglePlacesReadTask", "onPostExecute Exit");
 }
 private void ShowNearbyPlaces(List<HashMap<String, String>> 
nearbyPlacesList) {
 for (int i = 0; i < nearbyPlacesList.size(); i++) {
 Log.d("onPostExecute","Entered into showing locations");
 MarkerOptions markerOptions = new MarkerOptions();
 HashMap<String, String> googlePlace = nearbyPlacesList.get(i);
 double lat = Double.parseDouble(googlePlace.get("lat"));
 double lng = Double.parseDouble(googlePlace.get("lng"));
 String placeName = googlePlace.get("place_name");
 String vicinity = googlePlace.get("vicinity");
 LatLng latLng = new LatLng(lat, lng);
 markerOptions.position(latLng);
 markerOptions.title(placeName + " : " + vicinity);
 mMap.addMarker(markerOptions);
 
markerOptions.icon(BitmapDescriptorFactory.defaultMarker(BitmapDescriptorF
actory.HUE_RED));
 //move map camera
 mMap.moveCamera(CameraUpdateFactory.newLatLng(latLng));
 mMap.animateCamera(CameraUpdateFactory.zoomTo(11));
 }
 }
}
DownloadUrl.Java
package com.app.besmartybd.nearplace;
import android.util.Log;
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;
public class DownloadUrl {
 public String readUrl(String strUrl) throws IOException {
 String data = "";
 InputStream iStream = null;
 HttpURLConnection urlConnection = null;
 try {
 URL url = new URL(strUrl);
 // Creating an http connection to communicate with url
 urlConnection = (HttpURLConnection) url.openConnection();
 // Connecting to url
 urlConnection.connect();
 // Reading data from url
 iStream = urlConnection.getInputStream();
 BufferedReader br = new BufferedReader(new 
InputStreamReader(iStream));
 StringBuffer sb = new StringBuffer();
 String line = "";
 while ((line = br.readLine()) != null) {
 sb.append(line);
 }
 data = sb.toString();
 Log.d("downloadUrl", data.toString());
 br.close();
 } catch (Exception e) {
 Log.d("Exception", e.toString());
 } finally {
 iStream.close();
 urlConnection.disconnect();
 }
 return data;
 }
}
DataParser.Java
package com.app.besmartybd.nearplace;
import android.util.Log;
import org.json.JSONArray;
import org.json.JSONException;
import org.json.JSONObject;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
public class DataParser {
 public List<HashMap<String, String>> parse(String jsonData) {
 JSONArray jsonArray = null;
 JSONObject jsonObject;
 try {
 Log.d("Places", "parse");
 jsonObject = new JSONObject((String) jsonData);
 jsonArray = jsonObject.getJSONArray("results");
 } catch (JSONException e) {
 Log.d("Places", "parse error");
 e.printStackTrace();
 }
 return getPlaces(jsonArray);
 }
 private List<HashMap<String, String>> getPlaces(JSONArray jsonArray) {
 int placesCount = jsonArray.length();
 List<HashMap<String, String>> placesList = new ArrayList<>();
 HashMap<String, String> placeMap = null;
 Log.d("Places", "getPlaces");
 for (int i = 0; i < placesCount; i++) {
 try {
 placeMap = getPlace((JSONObject) jsonArray.get(i));
 placesList.add(placeMap);
Log.d("Places", "Adding places");
 } catch (JSONException e) {
 Log.d("Places", "Error in Adding places");
 e.printStackTrace();
 }
 }
 return placesList;
 }
 private HashMap<String, String> getPlace(JSONObject googlePlaceJson) {
 HashMap<String, String> googlePlaceMap = new HashMap<String, 
String>();
 String placeName = "-NA-";
 String vicinity = "-NA-";
 String latitude = "";
 String longitude = "";
 String reference = "";
 Log.d("getPlace", "Entered");
 try {
 if (!googlePlaceJson.isNull("name")) {
 placeName = googlePlaceJson.getString("name");
 }
 if (!googlePlaceJson.isNull("vicinity")) {
 vicinity = googlePlaceJson.getString("vicinity");
 }
 latitude = 
googlePlaceJson.getJSONObject("geometry").getJSONObject("location").getStr
ing("lat");
 longitude = 
googlePlaceJson.getJSONObject("geometry").getJSONObject("location").getStr
ing("lng");
 reference = googlePlaceJson.getString("reference");
 googlePlaceMap.put("place_name", placeName);
 googlePlaceMap.put("vicinity", vicinity);
 googlePlaceMap.put("lat", latitude);
 googlePlaceMap.put("lng", longitude);
 googlePlaceMap.put("reference", reference);
 Log.d("getPlace", "Putting Places");
 } catch (JSONException e) {
 Log.d("getPlace", "Error");
 e.printStackTrace();
 }
 return googlePlaceMap;
 }
}
Activity_maps.xml
<FrameLayout
 android:layout_width="match_parent"
 android:layout_height="match_parent"
 xmlns:android="http://schemas.android.com/apk/res/android">
 <fragment android:id="@+id/map"
 android:name="com.google.android.gms.maps.SupportMapFragment"
 xmlns:android="http://schemas.android.com/apk/res/android"
 xmlns:map="http://schemas.android.com/apk/res-auto"
 xmlns:tools="http://schemas.android.com/tools"
 android:layout_width="match_parent"
 android:layout_height="match_parent"
 
tools:context="com.androidtutorialpoint.googlemapsnearbyplaces.MapsActivit
y"/>
 <LinearLayout 
xmlns:android="http://schemas.android.com/apk/res/android"
 xmlns:tools="http://schemas.android.com/tools"
 android:layout_width="match_parent"
 android:layout_height="wrap_content"
 android:orientation="vertical">
 <Button
 android:id="@+id/btnRestaurant"
 android:layout_width="wrap_content"
 android:layout_height="wrap_content"
 android:text="Nearby Restaurants" />
 <Button
 android:id="@+id/btnHospital"
 android:layout_width="wrap_content"
 android:layout_height="wrap_content"
 android:text="Nearby Hospitals" />
 <Button
 android:id="@+id/btnSchool"
 android:layout_width="wrap_content"
 android:layout_height="wrap_content"
 android:text="Nearby Schools" />
 </LinearLayout>
</FrameLayout>
