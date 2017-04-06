# Location-Management
Identification of each building in certain area using co-ordinate value(latitude and longitude value) of a position which are stored in server.
This Android Project like such a map application. We have developed our application for a campus area to identify each building belonging to this campus. This application can be useful to the newer people who are much not familiar with this area.
This application will show user current building name from the phone. For that user should have android OS based phone with internet facility. The application must be installed on your android device. This application will show the user  current building location  with the co-ordinate value of user’s current position.

Here I add the java files and xml files(CustomList.java, ParseJSON.java, MainActivity.java, acitivity_main.xml, custom_listview.xml, AndroidManifest.xml) related to this project.

1. CustomList.java

package com.example.parvin.gps_ex1;
import android.app.Activity;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ArrayAdapter;
import android.widget.TextView;
public class CustomList extends ArrayAdapter<String>{

    private String[] locations;
    private String[] latitudes;
    private String[] longitudes;
    private Activity context;

    public CustomList(Activity context, String[] ids, String[] names, String[] emails) {
        super(context, R.layout.custom_listview, ids);
        this.context = context;
        this.locations = ids;
        this.latitudes = names;
        this.longitudes = emails;
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        LayoutInflater inflater = context.getLayoutInflater();
        View listViewItem = inflater.inflate(R.layout.custom_listview, null, true);
        TextView textViewId = (TextView) listViewItem.findViewById(R.id.txtId);
        TextView textViewName = (TextView) listViewItem.findViewById(R.id.txtName);
        TextView textViewEmail = (TextView) listViewItem.findViewById(R.id.txtEmail);

        textViewId.setText(locations[position]);
        textViewName.setText(latitudes[position]);
        textViewEmail.setText(longitudes[position]);

        return listViewItem;
    }
}


2. ParseJSON.java

package com.example.parvin.gps_ex1;
import org.json.JSONArray;
import org.json.JSONException;
import org.json.JSONObject;

public class ParseJSON {

    public static String[] locations;
    public static String[] latitudes;
    public static String[] longitudes;

    public static final String JSON_ARRAY = "result";
    public static final String KEY_ID = "location";
    public static final String KEY_NAME = "latitude";
    public static final String KEY_EMAIL = "longitude";

    private JSONArray users = null;
    private String json;

    public ParseJSON(String json){
        this.json = json;
    }

    protected void parseJSON(){
        JSONObject jsonObject=null;
        try {
            jsonObject = new JSONObject(json);
            users = jsonObject.getJSONArray(JSON_ARRAY);

            locations = new String[users.length()];
            latitudes = new String[users.length()];
            longitudes = new String[users.length()];

            for(int i=0;i<users.length();i++){
                JSONObject jo = users.getJSONObject(i);
                locations[i] = jo.getString(KEY_ID);
                latitudes[i] = jo.getString(KEY_NAME);
                longitudes[i] = jo.getString(KEY_EMAIL);
            }
        } catch (JSONException e) {
            e.printStackTrace();
        }
    }
}

3. MainActivity.java

package com.example.parvin.gps_ex1;
import android.Manifest;
import android.app.Activity;
import android.content.Context;
import android.content.pm.PackageManager;
import android.location.Location;
import android.location.LocationListener;
import android.location.LocationManager;
import android.os.Handler;
import android.support.design.widget.Snackbar;
import android.support.v4.app.ActivityCompat;
import android.support.v4.content.ContextCompat;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.TextView;
import android.widget.Toast;
import com.android.volley.RequestQueue;
import com.android.volley.Response;
import com.android.volley.VolleyError;
import com.android.volley.toolbox.StringRequest;
import com.android.volley.toolbox.Volley;

public class MainActivity extends AppCompatActivity implements LocationListener,View.OnClickListener{

    public static final String JSON_URL = "http://parvinandroid.96.lt/location.php";


    private Context context;
    private Activity activity;
    private static final int PERMISSION_REQUEST_CODE = 1;
    private static final int PERMISSION_REQUEST_CODE2 = 2;
    private View view;


    LocationManager locationManager;
    Handler handler;
    Location location;

    double latitude;
    double longitude;

    TextView lat;
    TextView lon;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        context = getApplicationContext();
        activity = this;

        handler = new Handler();

        lat = (TextView) findViewById(R.id.textView);
        lon = (TextView) findViewById(R.id.textView2);



         if (!checkPermission()){
            requestPermission();
        }
        else {

             locationManager = (LocationManager) this.getSystemService(Context.LOCATION_SERVICE);
             locationManager.requestLocationUpdates(LocationManager.NETWORK_PROVIDER, 0, 0, this);
             location = locationManager.getLastKnownLocation(LocationManager.NETWORK_PROVIDER);

             if (location != null){
                 latitude = location.getLatitude();
                 longitude = location.getLongitude();
             }

             sendRequest();


         }


    }

    public Runnable runLocation = new Runnable(){
        @Override
        public void run() {
            lat.setText(String.valueOf(latitude));
            lon.setText(String.valueOf(longitude));


            
            //Toast.makeText(MainActivity.this, "location: Lati: "+latitude+" Longi: "+longitude, Toast.LENGTH_SHORT).show();

            MainActivity.this.handler.postDelayed(MainActivity.this.runLocation, 5000);
            //sendRequest();

            Location user_loc;
            for(int i=0;i<ParseJSON.locations.length;i++){
                user_loc = new Location(ParseJSON.locations[i]);
                user_loc.setLatitude(Double.parseDouble(ParseJSON.latitudes[i]));
                user_loc.setLongitude(Double.parseDouble(ParseJSON.longitudes[i]));


                if (location != null) {
                    latitude = location.getLatitude();
                    longitude = location.getLongitude();
                    //Toast.makeText(MainActivity.this, "hello test", Toast.LENGTH_LONG).show();

                    int Distance_From_Location = 50;
                    if (location.distanceTo(user_loc) < Distance_From_Location) {
                        Toast.makeText(MainActivity.this, "Inside: " + ParseJSON.locations[i], Toast.LENGTH_LONG).show();
                    }
                }
            }

        }
    };

    private void sendRequest(){

        StringRequest stringRequest = new StringRequest(JSON_URL,
                new Response.Listener<String>() {
                    @Override
                    public void onResponse(String response) {
                        showJSON(response);
                    }
                },
                new Response.ErrorListener() {
                    @Override
                    public void onErrorResponse(VolleyError error) {
                        Toast.makeText(MainActivity.this,error.getMessage(), Toast.LENGTH_LONG).show();
                    }
                });

        RequestQueue requestQueue = Volley.newRequestQueue(this);
        requestQueue.add(stringRequest);
    }

    private void showJSON(String json){
        ParseJSON pj = new ParseJSON(json);
        pj.parseJSON();
        Location user_loc;
        for(int i=0;i<ParseJSON.locations.length;i++){
            user_loc = new Location(ParseJSON.locations[i]);
            user_loc.setLatitude(Double.parseDouble(ParseJSON.latitudes[i]));
            user_loc.setLongitude(Double.parseDouble(ParseJSON.longitudes[i]));

            locationManager = (LocationManager) this.getSystemService(Context.LOCATION_SERVICE);
            locationManager.requestLocationUpdates(LocationManager.NETWORK_PROVIDER, 0, 0, this);
            location = locationManager.getLastKnownLocation(LocationManager.NETWORK_PROVIDER);
            if (!checkPermission()){
                requestPermission();
            }
            if (location != null) {
                latitude = location.getLatitude();
                longitude = location.getLongitude();
                //Toast.makeText(MainActivity.this, "hello test", Toast.LENGTH_LONG).show();

                int Distance_From_Location = 50;
                if (location.distanceTo(user_loc) < Distance_From_Location) {
                    Toast.makeText(MainActivity.this, "Inside: " + ParseJSON.locations[i], Toast.LENGTH_LONG).show();
                }
            }
        }

        handler.postDelayed(runLocation, 1000);

    }

    @Override
    public void onLocationChanged(Location location) {
        latitude = location.getLatitude();
        longitude = location.getLongitude();
    }

    @Override
    public void onStatusChanged(String provider, int status, Bundle extras) {

    }

    @Override
    public void onProviderEnabled(String provider) {

    }

    @Override
    public void onProviderDisabled(String provider) {

    }

    private boolean checkPermission(){
        int result = ContextCompat.checkSelfPermission(context, Manifest.permission.ACCESS_FINE_LOCATION);
        int result2 = ContextCompat.checkSelfPermission(context, Manifest.permission.ACCESS_NETWORK_STATE);
        if (result == PackageManager.PERMISSION_GRANTED && result2 == PackageManager.PERMISSION_GRANTED){

            return true;

        } else {

            return false;

        }
    }

    private void requestPermission(){

        if (ActivityCompat.shouldShowRequestPermissionRationale(activity, Manifest.permission.ACCESS_FINE_LOCATION) && ActivityCompat.shouldShowRequestPermissionRationale(activity, Manifest.permission.ACCESS_NETWORK_STATE)){

            Toast.makeText(context,"GPS permission and Network allows us to access location data. Please allow in App Settings for additional functionality.",Toast.LENGTH_LONG).show();

        } else {

            ActivityCompat.requestPermissions(activity,new String[]{Manifest.permission.ACCESS_FINE_LOCATION},PERMISSION_REQUEST_CODE);
            ActivityCompat.requestPermissions(activity,new String[]{Manifest.permission.ACCESS_NETWORK_STATE},PERMISSION_REQUEST_CODE);
        }
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, String permissions[], int[] grantResults) {
        switch (requestCode) {
            case PERMISSION_REQUEST_CODE:
                if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {

                    Snackbar.make(view,"Permission Granted, Now you can access location data.",Snackbar.LENGTH_LONG).show();

                } else {

                    Snackbar.make(view,"Permission Denied, You cannot access location data.",Snackbar.LENGTH_LONG).show();

                }
                break;

            case PERMISSION_REQUEST_CODE2:
                if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {

                    Snackbar.make(view,"Permission Granted, Now you can access Network data.",Snackbar.LENGTH_LONG).show();

                } else {

                    Snackbar.make(view,"Permission Denied, You cannot access Network data.",Snackbar.LENGTH_LONG).show();

                }
                break;
        }
    }

    @Override
    public void onClick(View v) {
        sendRequest();
        handler.postDelayed(runLocation, 1000);

    }
}


4. activity_main.xml

<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/activity_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context="com.example.jaydeep.gps_ex1.MainActivity">

    <TextView
        android:text="TextView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerHorizontal="true"
        android:id="@+id/textView" />

    <TextView
        android:text="TextView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@+id/textView"
        android:layout_alignStart="@+id/textView"
        android:layout_marginTop="92dp"
        android:id="@+id/textView2" />

    <Button
        android:text="Show Current Location"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerVertical="true"
        android:layout_centerHorizontal="true"
        android:id="@+id/button" />
</RelativeLayout>


5. custom_listview.xml

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="match_parent">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="txtId"
        android:id="@+id/txtId"
        android:layout_gravity="center_horizontal"
        android:textStyle="bold"
        android:textSize="26dp" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="txtname"
        android:id="@+id/txtName"
        android:layout_gravity="center_horizontal"
        android:textStyle="bold"
        android:textSize="26dp" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="txtEmail"
        android:id="@+id/txtEmail"
        android:layout_gravity="center_horizontal"
        android:textStyle="bold"
        android:textSize="26dp" />
</LinearLayout>


6. AndriodManifest.xml

<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.parvin.gps_ex1">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.INTERNET"/>

</manifest>
