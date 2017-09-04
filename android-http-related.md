## Http request

> OkHttp

```java
package com.xyz.ray.pushapp;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.Toast;

import java.io.IOException;

import cn.jpush.android.api.JPushInterface;
import okhttp3.Call;
import okhttp3.Callback;
import okhttp3.OkHttpClient;
import okhttp3.Request;
import okhttp3.Response;

public class MainActivity extends AppCompatActivity {
    private OkHttpClient client = null;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        JPushInterface.setDebugMode(true);
        JPushInterface.init(this);
        regisBtnHandlers();
        init();
    }

    private void init() {
        client = new OkHttpClient();
    }

    private void regisBtnHandlers() {
        Button getDataBtn = (Button) findViewById(R.id.get_data_btn);
        getDataBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Toast.makeText(getApplicationContext(), "You have clicked on the button!!!", Toast.LENGTH_SHORT).show();
                Log.i("http", "Before http request.");
                getData();
                Log.i("http", "After http request.");
            }
        });
    }

    private void getData() {
//        String uri = "http://api.shendupeiban.com/comm/v1/token";
//        String uri = "http://192.168.1.240:8111/classcard/getClassInfo/5758225196885131327";
        String uri = "https://github.com/sereneray/android-useful-code/blob/master/http-related.md";
        try {
            Request request = new Request.Builder().url(uri).build();
            client.newCall(request).enqueue(new Callback() {
                @Override
                public void onFailure(Call call, IOException e) {
                    e.printStackTrace();
                }

                @Override
                public void onResponse(Call call, final Response response) throws IOException {
                    Log.i("http", "Http status code => " + response.code());
                    if (response.code() == 200) {
                        Log.i("http", response.body().string());
                    } else {
                        Toast.makeText(getApplicationContext(), "Request error!", Toast.LENGTH_SHORT).show();
                    }
                    if (!response.isSuccessful()) {
                        Toast.makeText(getApplicationContext(), "Connect to server error", Toast.LENGTH_SHORT).show();
                        throw new IOException("Unexpected code " + response);
                    }
                }
            });
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}


```
