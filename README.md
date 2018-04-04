# RealmCRUD
#CRUD with realm database

##### Step 1) Update build.gradle

Tambahkan dependencies pada build.gradle dibawah ini.

```java
dependencies {
    ...
    ...
    //Tambahkan ini
    compile 'io.realm:realm-android:0.87.2'
    //Untuk menampilkan data dgn cardview
    compile 'com.android.support:design:23.1.1'
    compile 'com.android.support:cardview-v7:23.1.1'
}
```
##### Step 2) Membuat BaseApp.java

Kita harus membuat konfigurasi default Realm ke dalam class yang meng-extends Application. Berikut contoh kode lengkapnya.
```java

import io.realm.DynamicRealm;
import io.realm.Realm;
import io.realm.RealmConfiguration;
import io.realm.RealmMigration;
import io.realm.RealmSchema;


public class BaseApp extends Application {

    @Override
    public void onCreate() {
        super.onCreate();

        //kode konfigurasi Realm
        RealmConfiguration config = new RealmConfiguration.Builder(this)
                //versi database
                .schemaVersion(0)
                .migration(new DataMigration())
                .build();

        Realm.setDefaultConfiguration(config);

    }

    private class DataMigration implements RealmMigration {
        @Override
        public void migrate(DynamicRealm realm, long oldVersion, long newVersion) {

            //Mengambil schema
            RealmSchema schema = realm.getSchema();

            //membuat schema baru jika versi 0
            if (oldVersion == 0) {
                schema.create("Article")
                        .addField("id", int.class)
                        .addField("title", String.class)
                        .addField("description", String.class);
                oldVersion++;
            }

        }
    }
}
```
##### Step 3) Membuat Class Article.java

Selanjutnya kita membuat class Article sebagai Realm data models dengan cara mengextends pada RealmObject. Field id akan kita gunakan sebagai Primary Key. Tujuan Class ini akan kita gunakan untuk memanajemen data yang kita buat seperti menambah dan menghapus data dengan memanfaatkan getter dan setter. Ada cara cepat untuk membuat getter dan setter, yaitu dengan klik kanan dalam editor kemudian pilih Generate -> Getter and Setter, sebelum melakukannya tulislah dulu variable dalam kelas tersebut yang nantinya akan di generate getter dan setter nya. Berikut kode lengkap-nya
```java
package com.gookkis.realmtutorial.helper;

import io.realm.RealmObject;
import io.realm.annotations.PrimaryKey;

public class Article extends RealmObject {
    @PrimaryKey
    private int id;
    private String title;
    private String description;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getTitle() {
        return title;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    public String getDescription() {
        return description;
    }
}
```
##### Step 4) Membuat Class ArticleModel.java

Buatlah class ArticleModel.java seperti kode dibawah yang akan digunakan sebagai penampun object sementara yang akan ditampung sebagai ArrayList yang nantinya akan membantu dalam operasi CRUD.
```java

public class ArticleModel {
    private int id;
    private String title;
    private String description;
 
 
    public ArticleModel(int id, String title, String description) {
        this.id = id;
        this.title = title;
        this.description = description;
    } 
 
 
    public int getId() { 
        return id;
    } 
 
 
    public void setId(int id) {
        this.id = id;
    } 
 
 
    public String getTitle() {
        return title;
    } 
 
 
    public void setTitle(String title) {
        this.title = title;
    } 
 
 
    public String getDescription() {
        return description;
    } 
 
 
    public void setDescription(String description) {
        this.description = description;
    } 
}
```
##### Step 5) Membuat Class Helper.java

Kelas ini merupakan class helper yang berisi method-method yang akan digunakan untuk CRUD terhadap Realm yang kita buat pada saat kita membuat BaseApp. Berikut kode lengkapnya dengan comment penjelasan.
```java
import android.content.Context;
import android.util.Log;
import android.widget.Toast;

import com.gookkis.realmtutorial.model.ArticleModel;

import java.util.ArrayList;

import io.realm.Realm;
import io.realm.RealmResults;
import io.realm.Sort;


public class RealmHelper {


    private static final String TAG = "RealmHelper";


    private Realm realm;
    private RealmResults<Article> realmResult;
    public Context context;

    /**
     * constructor untuk membuat instance realm
     *
     * @param context
     */
    public RealmHelper(Context context) {
        realm = Realm.getInstance(context);
        this.context = context;
    }


    /**
     * menambah data
     *
     * @param title
     * @param description
     */
    public void addArticle(String title, String description) {
        Article article = new Article();
        article.setId((int) (System.currentTimeMillis() / 1000));
        article.setTitle(title);
        article.setDescription(description);


        realm.beginTransaction();
        realm.copyToRealm(article);
        realm.commitTransaction();


        showLog("Added ; " + title);
        showToast(title + " berhasil disimpan.");
    }


    /**
     * method mencari semua article
     */
    public ArrayList<ArticleModel> findAllArticle() {
        ArrayList<ArticleModel> data = new ArrayList<>();


        realmResult = realm.where(Article.class).findAll();
        realmResult.sort("id", Sort.DESCENDING);
        if (realmResult.size() > 0) {
            showLog("Size : " + realmResult.size());


            for (int i = 0; i < realmResult.size(); i++) {
                String title, description;
                int id = realmResult.get(i).getId();
                title = realmResult.get(i).getTitle();
                description = realmResult.get(i).getDescription();
                data.add(new ArticleModel(id, title, description));
            }

        } else {
            showLog("Size : 0");
            showToast("Database Kosong!");
        }

        return data;
    }


    /**
     * method update article
     *
     * @param id
     * @param title
     * @param description
     */
    public void updateArticle(int id, String title, String description) {
        realm.beginTransaction();
        Article article = realm.where(Article.class).equalTo("id", id).findFirst();
        article.setTitle(title);
        article.setDescription(description);
        realm.commitTransaction();
        showLog("Updated : " + title);

        showToast(title + " berhasil diupdate.");
    }


    /**
     * method menghapus article berdasarkan id
     *
     * @param id
     */
    public void deleteData(int id) {
        RealmResults<Article> dataDesults = realm.where(Article.class).equalTo("id", id).findAll();
        realm.beginTransaction();
        dataDesults.remove(0);
        dataDesults.removeLast();
        dataDesults.clear();
        realm.commitTransaction();

        showToast("Hapus data berhasil.");
    }


    /**
     * membuat log
     *
     * @param s
     */
    private void showLog(String s) {
        Log.d(TAG, s);

    }

    /**
     * Membuat Toast Informasi
     */
    private void showToast(String s) {
        Toast.makeText(context, s, Toast.LENGTH_LONG).show();
    }
}
```
Yang paling penting dalam pembuatan class helper ini adalah membuat method dalam class helper ini kita harus selalu memperhatikan method realm.beginTransaction(); karena dalam mengakses database realm kita perlu membukanya kemudian melakukan CRUD atau lainnya setelah itu kita harus menutup kembali dengan method realm.commitTransaction();. Berikut penjelasan dalam kode.

```java
   realm.beginTransaction();

   //masukan kode anda disini

   realm.commitTransaction();
```
##### Step 6) Membuat Layout

Sebelum kita membuat Class activity kita akan membuat 3 layout yaitu untuk daftar list/data, tambah data, dan edit data namun  saya akan membuat 4 layout dimana satu layout merupakan child dari layout daftar list. Layout tambah dan edit saya jadikan satu sedangkan satu lagi adalah layout item_article yang akan kita jadikan inflate untuk menampilkan data dalam list. Berikut ini kode-kode nya
```xml
activity_main.xml

<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#eeeeee"
    android:fitsSystemWindows="true"
    tools:context=".activity.HomeActivity">


    <android.support.design.widget.AppBarLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:theme="@style/AppTheme.AppBarOverlay">


        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            android:background="?attr/colorPrimary"
            app:popupTheme="@style/AppTheme.PopupOverlay" />


    </android.support.design.widget.AppBarLayout>


    <include layout="@layout/content_home" />


    <android.support.design.widget.FloatingActionButton
        android:id="@+id/fab"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="bottom|end"
        android:layout_margin="@dimen/activity_horizontal_margin"
        android:src="@drawable/ic_add_circle_outline" />


</android.support.design.widget.CoordinatorLayout>
```

```xml
activity_tambah.xml

<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context=".activity.AddActivity">


    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">


        <EditText
            android:id="@+id/inputTitle"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginBottom="10dp"
            android:hint="Title" />


        <EditText
            android:id="@+id/inputDescription"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="Description" />


        <LinearLayout
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:layout_marginTop="20dp"
            android:orientation="horizontal">


            <Button
                android:id="@+id/delete"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_gravity="center"
                android:text="Delete"
                android:visibility="gone"/>


            <Button
                android:id="@+id/save"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="save" />
        </LinearLayout>
    </LinearLayout>


</RelativeLayout>
```
```xml
content_home.xml (child activity_main.xml)

<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:layout_behavior="@string/appbar_scrolling_view_behavior"
    tools:context=".activity.HomeActivity"
    tools:showIn="@layout/activity_main">


    <android.support.v7.widget.RecyclerView
        android:id="@+id/rvArticle"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />


</RelativeLayout>
```
```xml
item_article.xml

<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:layout_behavior="@string/appbar_scrolling_view_behavior"
    tools:context=".activity.HomeActivity"
    tools:showIn="@layout/activity_main">


    <android.support.v7.widget.RecyclerView
        android:id="@+id/rvArticle"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />


</RelativeLayout>
```
##### Step 7) Membuat Class Adapter dan Activity

Kita akan membuat class yang akan digunakan sebagai class adapter dan activity. Untuk kelas activity kita akan membuat 3 class yaitu MainActivity.java, AddActivity.java, dan EditActivity.java , dalam kode sudah saya tambahkan komentar untuk penjelasan atas kode-kode yang ada. Berikut ketiga class tersebut.

MainActivity.java

```java

import android.content.Intent;
import android.os.Bundle;
import android.support.design.widget.FloatingActionButton;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.LinearLayoutManager;
import android.support.v7.widget.RecyclerView;
import android.support.v7.widget.Toolbar;
import android.view.View;

import com.gookkis.realmtutorial.R;
import com.gookkis.realmtutorial.adapter.AdapterArticle;
import com.gookkis.realmtutorial.helper.RealmHelper;
import com.gookkis.realmtutorial.model.ArticleModel;

import java.util.ArrayList;

public class MainActivity extends AppCompatActivity {


    private static final String TAG = "HomeActivity";


    private RecyclerView recyclerView;
    private RealmHelper helper;
    private ArrayList<ArticleModel> data;


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);


        data = new ArrayList<>();
        helper = new RealmHelper(MainActivity.this);


        recyclerView = (RecyclerView) findViewById(R.id.rvArticle);
        recyclerView.setLayoutManager(new LinearLayoutManager(this));


        FloatingActionButton fab = (FloatingActionButton) findViewById(R.id.fab);
        fab.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                startActivity(new Intent(getApplicationContext(), AddActivity.class));
                finish();
            }
        });


        setRecyclerView();
    }


    /**
     * set recyclerview with try get data from realm
     */
    public void setRecyclerView() {
        try {
            data = helper.findAllArticle();
        } catch (Exception e) {
            e.printStackTrace();
        }
        AdapterArticle adapter = new AdapterArticle(data, new AdapterArticle.OnItemClickListener() {
            @Override
            public void onClick(ArticleModel item) {
                Intent i = new Intent(getApplicationContext(), EditActivity.class);
                i.putExtra("id", item.getId());
                i.putExtra("title", item.getTitle());
                i.putExtra("description", item.getDescription());
                startActivity(i);
                finish();
            }
        });
        recyclerView.setAdapter(adapter);
    }


    @Override
    protected void onResume() {
        super.onResume();
        try {
            data = helper.findAllArticle();
        } catch (Exception e) {
            e.printStackTrace();
        }
        //data = helper.findAllArticle();
        setRecyclerView();
    }
}
```
AddActivity.java
```java

import android.content.Intent;
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;

import com.gookkis.realmtutorial.R;
import com.gookkis.realmtutorial.helper.RealmHelper;

public class AddActivity extends AppCompatActivity {


    private RealmHelper realmHelper;
    private EditText inputDescription;
    private EditText inputTitle;
    private Button save;


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_tambah);


        realmHelper = new RealmHelper(AddActivity.this);


        inputTitle = (EditText) findViewById(R.id.inputTitle);
        inputDescription = (EditText) findViewById(R.id.inputDescription);
        save = (Button) findViewById(R.id.save);


        save.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                //inisialisasi string
                String title, description;

                //mengambil text dr edittext
                title = inputTitle.getText().toString();
                description = inputDescription.getText().toString();

                //menambahkan data pada database
                realmHelper.addArticle(title, description);

                //menutup Add Activity
                finish();
                //kembali ke MainActivity
                Intent i = new Intent(AddActivity.this, MainActivity.class);
                startActivity(i);
            }
        });
    }
}
```
 EditActivity.java
```java

import android.content.Intent;
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;

import com.gookkis.realmtutorial.R;
import com.gookkis.realmtutorial.helper.RealmHelper;
import com.gookkis.realmtutorial.model.ArticleModel;

import java.util.ArrayList;

public class EditActivity extends AppCompatActivity {


    private int position;
    private Button delete, save;
    private EditText inputTitle, inputDescription;
    private RealmHelper helper;
    private String title, description;
    private String intentTitle, intentDescription;
    private ArrayList<ArticleModel> data;


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_tambah);

        helper = new RealmHelper(EditActivity.this);
        data = new ArrayList<>();
        position = getIntent().getIntExtra("id", 0);
        intentTitle = getIntent().getStringExtra("title");
        intentDescription = getIntent().getStringExtra("description");


        delete = (Button) findViewById(R.id.delete);
        save = (Button) findViewById(R.id.save);


        inputTitle = (EditText) findViewById(R.id.inputTitle);
        inputDescription = (EditText) findViewById(R.id.inputDescription);


        inputTitle.setText(intentTitle);
        inputDescription.setText(intentDescription);


        delete.setVisibility(View.VISIBLE);

        //perintah untuk menghapus
        delete.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                //menghapus data dari database
                helper.deleteData(position);

                //berpindah ke MainActivity
                startActivity(new Intent(EditActivity.this, MainActivity.class));
                finish();
            }
        });

        //perintah mengupdate data
        save.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {

                //mengambil text dari edittext
                title = inputTitle.getText().toString();
                description = inputDescription.getText().toString();

                //melakukan update artikel
                helper.updateArticle(position, title, description);

                //berpindah ke MainActivity
                startActivity(new Intent(EditActivity.this, MainActivity.class));
                finish();
            }
        });


    }


}
```


Kemudian kita akan membuat class adapter yang akan digunakan untuk menampilkan data yang berasal dari realm database dalam bentuk ArrayList pada sebuah RecyclerView dengan meng-inflate item_article.xml sehingga berbentuk custom listview. Di dalam item_article.xml kita menggunakan CardView agar tampilan lebih menarik. Berikut kode class adapter.

```java
import android.support.v7.widget.RecyclerView;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;

import com.gookkis.realmtutorial.R;
import com.gookkis.realmtutorial.model.ArticleModel;

import java.util.ArrayList;

public class AdapterArticle extends RecyclerView.Adapter<AdapterArticle.ViewHolder> {


    private final OnItemClickListener listener;
    private ArrayList<ArticleModel> article;


    public AdapterArticle(ArrayList<ArticleModel> article, OnItemClickListener listener) {
        this.article = article;
        this.listener = listener;
    }


    @Override
    public AdapterArticle.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        View view = LayoutInflater.from(parent.getContext()).inflate(R.layout.item_article, null);
        ViewHolder vh = new ViewHolder(view);
        return vh;
    }


    @Override
    public void onBindViewHolder(AdapterArticle.ViewHolder holder, int position) {
        holder.click(article.get(position), listener);
        holder.tvId.setText(String.valueOf(article.get(position).getId()));
        holder.title.setText(article.get(position).getTitle());
        holder.description.setText(article.get(position).getDescription());
    }


    @Override
    public int getItemCount() {
        return article.size();
    }


    public class ViewHolder extends RecyclerView.ViewHolder {
        TextView tvId, title, description;


        public ViewHolder(View itemView) {
            super(itemView);
            tvId = (TextView) itemView.findViewById(R.id.tvId);
            title = (TextView) itemView.findViewById(R.id.tvTitle);
            description = (TextView) itemView.findViewById(R.id.tvDescription);
        }


        public void click(final ArticleModel articleModel, final OnItemClickListener listener) {
            itemView.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    listener.onClick(articleModel);
                }
            });
        }
    }


    public interface OnItemClickListener {
        void onClick(ArticleModel item);
    }


} 
```
##### Step 7) Update AndroidManifest.xml

Tambahkan kode berikut pada dalam AndroidManifest.xml.

```xml

    <application
        android:name=".BaseApp"
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/AppTheme.NoActionBar">
        <activity android:name=".activity.MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        <activity
            android:name=".activity.AddActivity"
            android:label="Tambah Data"
            android:theme="@style/AppTheme" />
        <activity
            android:name=".activity.EditActivity"
            android:label="Edit Data"
            android:theme="@style/AppTheme" />
    </application>

```
##### Step 8) Pastikan style.xml

Pastikan style.xml dalam folder values seperti dibawah ini.
```xml
<resources>

    <!-- Base application theme. -->
    <style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
    </style>

    <style name="AppTheme.NoActionBar" parent="Theme.AppCompat.Light.NoActionBar">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
    </style>

    <style name="AppTheme.AppBarOverlay" parent="ThemeOverlay.AppCompat.Dark.ActionBar" />


    <style name="AppTheme.PopupOverlay" parent="ThemeOverlay.AppCompat.Light" />

</resources>
```



