# NotePad
This is an AndroidStudio rebuild of google SDK sample NotePad
**一、基本功能实现**

1. 显示时间戳：

   效果截图：

   图一

   添加时间戳的位置在主页面的每个列表项中添加，即在notelist_item.xml布局文件中添加一个TextView

   关键代码：

   ```
   <TextView
           android:id="@android:id/text1"
           android:layout_width="257dp"
           android:layout_height="?android:attr/listPreferredItemHeight"
           android:gravity="center_vertical"
           android:paddingLeft="5dip"
           android:singleLine="true"
           android:textAppearance="?android:attr/textAppearanceLarge" />
   ```

   在NoteList类的PROJECTION中添加COLUMN_NAME_MODIFICATION_DATE字段

   关键代码：

   ```
   private static final String[] PROJECTION = new String[] {
               NotePad.Notes._ID, // 0
               NotePad.Notes.COLUMN_NAME_TITLE, // 1
               NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, //日期
               NotePad.Notes.COLUMN_NAME_TEXT_COLOR,
               NotePad.Notes.COLUMN_NAME_TEXT_SIZE,
               NotePad.Notes.COLUMN_NAME_BACK_COLOR,
       };
   ```

   数据库代码：

   ```
     @Override
          public void onCreate(SQLiteDatabase db) {
              db.execSQL("CREATE TABLE " + NotePad.Notes.TABLE_NAME + " ("
                      + NotePad.Notes._ID + " INTEGER PRIMARY KEY,"
                      + NotePad.Notes.COLUMN_NAME_TITLE + " TEXT,"
                      + NotePad.Notes.COLUMN_NAME_NOTE + " TEXT,"
                      + NotePad.Notes.COLUMN_NAME_CREATE_DATE + " INTEGER,"
                      + NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + " INTEGER"
                      + NotePad.Notes.COLUMN_NAME_TEXT_COLOR + "INTEGER" //新增加的修改字体颜色
                      + NotePad.Notes.COLUMN_NAME_TEXT_SIZE + "INTEGER" //新增加的修改字体大小
                      + NotePad.Notes.COLUMN_NAME_BACK_COLOR + "INTEGER" //颜色
                      + ");");
          }
   ```

   在dataColumns,viewIDs这两个参数中加入时间

   关键代码：

   ```
   String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,
                   NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE} ;
   
           // The view IDs that will display the cursor columns, initialized to the TextView in
           // noteslist_item.xml
           int[] viewIDs = { android.R.id.text1,
                   R.id.text1_date};
   ```

   ```
   SimpleCursorAdapter adapter
                   = new SimpleCursorAdapter(
                   this,                             // The Context for the ListView
                   R.layout.noteslist_item,          // Points to the XML for a list item
                   cursor,                           // The cursor to get items from
                   dataColumns,
                   viewIDs
           );
   ```

   

   

   2.查询功能：

   效果截图：

   图234

   在list_options_menu.xml中新建一个查询的按钮

   关键代码：

   ```
    <item
           android:id="@+id/menu_search"
           android:icon="@android:drawable/ic_menu_search"
           android:title="Search"
           android:showAsAction="always" />
   ```

   新建一个查找笔记内容的布局文件notesearch.xml

   代码：

   ```
   <?xml version="1.0" encoding="utf-8"?>
   <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
       android:layout_width="match_parent"
       android:layout_height="match_parent"
       android:orientation="vertical">
       <SearchView
           android:id="@+id/search_view"
           android:layout_width="match_parent"
           android:layout_height="wrap_content"
           android:iconifiedByDefault="false"
           android:queryHint="请输入搜索内容..."
           />
       <ListView
           android:id="@+id/list_view"
           android:layout_width="match_parent"
           android:layout_height="wrap_content"
           />
   
   </LinearLayout>
   ```

   在NoteList类中的onOptionsItemSelected方法中添加search查询的处理

   关键代码：

   ```
    case R.id.menu_search:
               /*
                   查找功能
                */
                   Intent intent = new Intent(this, NoteSearch.class);
                   this.startActivity(intent);
                   return true;
   ```

   新建一个NoteSearch类

   ```
   package com.example.android.notepad;
   import android.app.Activity;
   import android.content.Intent;
   import android.database.Cursor;
   import android.database.sqlite.SQLiteDatabase;
   import android.os.Bundle;
   import android.widget.ListView;
   import android.widget.SearchView;
   import android.widget.SimpleCursorAdapter;
   import android.widget.Toast;
   
   public class NoteSearch extends Activity implements SearchView.OnQueryTextListener
   {
       ListView listView;
       SQLiteDatabase sqLiteDatabase;
       /**
        * The columns needed by the cursor adapter
        */
       private static final String[] PROJECTION = new String[]{
               NotePad.Notes._ID, // 0
               NotePad.Notes.COLUMN_NAME_TITLE, // 1
               NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE//时间
       };
   
       public boolean onQueryTextSubmit(String query) {
           Toast.makeText(this, "您选择的是："+query, Toast.LENGTH_SHORT).show();
           return false;
       }
   
       @Override
       protected void onCreate(Bundle savedInstanceState) {
           super.onCreate(savedInstanceState);
           setContentView(R.layout.notesearch);
           SearchView searchView = findViewById(R.id.search_view);
           Intent intent = getIntent();
           if (intent.getData() == null) {
               intent.setData(NotePad.Notes.CONTENT_URI);
           }
           listView = findViewById(R.id.list_view);
           sqLiteDatabase = new NotePadProvider.DatabaseHelper(this).getReadableDatabase();
           //设置该SearchView显示搜索按钮
           searchView.setSubmitButtonEnabled(true);
   
           //设置该SearchView内默认显示的提示文本
           searchView.setQueryHint("查找");
           searchView.setOnQueryTextListener(this);
   
       }
       public boolean onQueryTextChange(String string) {
           String selection1 = NotePad.Notes.COLUMN_NAME_TITLE+" like ? or "+NotePad.Notes.COLUMN_NAME_NOTE+" like ?";
           String[] selection2 = {"%"+string+"%","%"+string+"%"};
           Cursor cursor = sqLiteDatabase.query(
                   NotePad.Notes.TABLE_NAME,
                   PROJECTION, // The columns to return from the query
                   selection1, // The columns for the where clause
                   selection2, // The values for the where clause
                   null,          // don't group the rows
                   null,          // don't filter by row groups
                   NotePad.Notes.DEFAULT_SORT_ORDER // The sort order
           );
           // The names of the cursor columns to display in the view, initialized to the title column
           String[] dataColumns = {
                   NotePad.Notes.COLUMN_NAME_TITLE,
                   NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE
           } ;
           // The view IDs that will display the cursor columns, initialized to the TextView in
           // noteslist_item.xml
           int[] viewIDs = {
                   android.R.id.text1,
                   android.R.id.text2
           };
           // Creates the backing adapter for the ListView.
           SimpleCursorAdapter adapter
                   = new SimpleCursorAdapter(
                   this,                             // The Context for the ListView
                   R.layout.noteslist_item,         // Points to the XML for a list item
                   cursor,                           // The cursor to get items from
                   dataColumns,
                   viewIDs
           );
           // Sets the ListView's adapter to be the cursor adapter that was just created.
           listView.setAdapter(adapter);
           return true;
       }
   }
   ```

   在注册表AndroidManifest.xml里面注册NoteSearch

   ```
   <intent-filter>
                   <action android:name="android.intent.action.NoteSearch" />
                   <action android:name="android.intent.action.SEARCH" />
   ```







**二、附加功能实现**    

1、在文本编辑界面中添加改变字体大小和字体颜色的功能

效果图：

图567

在editor_options_menu.xml中添加

关键代码：

```
<item
        android:id="@+id/font_size"
        android:title="字体大小">
        <!--子菜单-->
        <menu>
            <!--定义一组单选菜单项-->
            <group>
                <!--定义多个菜单项-->
                <item
                    android:id="@+id/font_10"
                    android:title="font10"
                    />

                <item
                    android:id="@+id/font_16"
                    android:title="font16" />
                <item
                    android:id="@+id/font_20"
                    android:title="font20" />
            </group>
        </menu>
    </item>

    <item
        android:title="颜色"
        android:id="@+id/font_color"
        >
        <menu>
            <!--定义一组普通菜单项-->
            <group>
                <!--定义两个菜单项-->
                <item
                    android:id="@+id/red_font"
                    android:title="red_title" />
                <item
                    android:title="black_title"
                    android:id="@+id/black_font"/>
            </group>
        </menu>
    </item>
```

在NoteEditor的onOptionsItemSelected(MenuItem item)中添加相应的case来响应事件

```
case R.id.font_10:
            mText.setTextSize(20);
            Toast toast =Toast.makeText(NoteEditor.this,"修改成功", Toast.LENGTH_SHORT);
            toast.show();
            break;
        case R.id.font_16:
                mText.setTextSize(32);
                Toast toast2 =Toast.makeText(NoteEditor.this,"修改成功",Toast.LENGTH_SHORT);
                toast2.show();
                break;
        case R.id.font_20:
            mText.setTextSize(40);
            Toast toast3 =Toast.makeText(NoteEditor.this,"修改成功",Toast.LENGTH_SHORT);
            toast3.show();
            break;
        case R.id.red_font:
            mText.setTextColor(Color.RED);
            Toast toast4 =Toast.makeText(NoteEditor.this,"修改成功",Toast.LENGTH_SHORT);
            toast4.show();
            break;
        case R.id.black_font:
            mText.setTextColor(Color.BLACK);
            Toast toast5 =Toast.makeText(NoteEditor.this,"修改成功",Toast.LENGTH_SHORT);
            toast5.show();
            break;
```

在数据库中加入相应的参数来保存字体大小和颜色的信息，可以在再次打开时显示修改后的字体

```
 + NotePad.Notes.COLUMN_NAME_TEXT_COLOR + "INTEGER" //新增加的修改字体颜色
                   + NotePad.Notes.COLUMN_NAME_TEXT_SIZE + "INTEGER" //新增加的修改字体大小
```





2、修改背景颜色，并可在主页面显示

效果图：

图89 10

数据库表地方添加颜色的字段

```
 + NotePad.Notes.COLUMN_NAME_BACK_COLOR + "INTEGER" //颜色
```

定义契约类

```
 public static final int DEFAULT_COLOR = 0; //白
    public static final int YELLOW_COLOR = 1; //黄
    public static final int BLUE_COLOR = 2; //蓝
    public static final int GREEN_COLOR = 3; //绿
    public static final int RED_COLOR = 4; //红
```

在NotePadProvider中添加对其相应的处理

```
sNotesProjectionMap.put(
                NotePad.Notes.COLUMN_NAME_BACK_COLOR,
                NotePad.Notes.COLUMN_NAME_BACK_COLOR);
```

insert中添加

```
if (values.containsKey(NotePad.Notes.COLUMN_NAME_BACK_COLOR) == false) {
            values.put(NotePad.Notes.COLUMN_NAME_BACK_COLOR, NotePad.Notes.DEFAULT_COLOR);
        }
```

在NoteEditor中添加，将颜色填充到ListView

```
public static class MyCursorAdapter extends SimpleCursorAdapter {
        public MyCursorAdapter(Context context, int layout, Cursor c,
                               String[] from, int[] to) {
            super(context, layout, c, from, to);
        }
        @Override
        public void bindView(View view, Context context, Cursor cursor){
            super.bindView(view, context, cursor);
            //从数据库中读取的cursor中获取笔记列表对应的颜色数据，并设置笔记颜色
            int x = cursor.getInt(cursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_BACK_COLOR));
            /**
             * 白 255 255 255
             * 黄 247 216 133
             * 蓝 165 202 237
             * 绿 161 214 174
             * 红 244 149 133
             */
            switch (x){
                case NotePad.Notes.DEFAULT_COLOR:
                    view.setBackgroundColor(Color.rgb(255, 255, 255));
                    break;
                case NotePad.Notes.YELLOW_COLOR:
                    view.setBackgroundColor(Color.rgb(247, 216, 133));
                    break;
                case NotePad.Notes.BLUE_COLOR:
                    view.setBackgroundColor(Color.rgb(165, 202, 237));
                    break;
                case NotePad.Notes.GREEN_COLOR:
                    view.setBackgroundColor(Color.rgb(161, 214, 174));
                    break;
                case NotePad.Notes.RED_COLOR:
                    view.setBackgroundColor(Color.rgb(244, 149, 133));
                    break;
                default:
                    view.setBackgroundColor(Color.rgb(255, 255, 255));
                    break;
            }
        }
    }

```

NoteList中的PROJECTION添加颜色项：

```
private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, //日期
            NotePad.Notes.COLUMN_NAME_TEXT_COLOR,
            NotePad.Notes.COLUMN_NAME_TEXT_SIZE,
            NotePad.Notes.COLUMN_NAME_BACK_COLOR,
    };
```

在NoteEditor的PROJECTION中添加

```
 private static final String[] PROJECTION =
        new String[] {
            NotePad.Notes._ID,
            NotePad.Notes.COLUMN_NAME_TITLE,
            NotePad.Notes.COLUMN_NAME_NOTE,
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,
            NotePad.Notes.COLUMN_NAME_TEXT_COLOR,
            NotePad.Notes.COLUMN_NAME_TEXT_SIZE,
            NotePad.Notes.COLUMN_NAME_BACK_COLOR
    };
```

在NoteEditor的onResume()中添加，将从数据库读取颜色并设置编辑界面背景色操作放入其中

```
int x = mCursor.getInt(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_BACK_COLOR));
        /**
         * 白 255 255 255
         * 黄 247 216 133
         * 蓝 165 202 237
         * 绿 161 214 174
         * 红 244 149 133
         */
        switch (x){
            case NotePad.Notes.DEFAULT_COLOR:
                mText.setBackgroundColor(Color.rgb(255, 255, 255));
                break;
            case NotePad.Notes.YELLOW_COLOR:
                mText.setBackgroundColor(Color.rgb(247, 216, 133));
                break;
            case NotePad.Notes.BLUE_COLOR:
                mText.setBackgroundColor(Color.rgb(165, 202, 237));
                break;
            case NotePad.Notes.GREEN_COLOR:
                mText.setBackgroundColor(Color.rgb(161, 214, 174));
                break;
            case NotePad.Notes.RED_COLOR:
                mText.setBackgroundColor(Color.rgb(244, 149, 133));
                break;
            default:
                mText.setBackgroundColor(Color.rgb(255, 255, 255));
                break;
```

先在菜单文件editor_options_menu.xml中添加一个更改背景的选项

```
<item android:id="@+id/menu_color"
        android:title="color"
        android:icon="@drawable/background"
        android:showAsAction="always"/>
```

在NoteEditor中找到onOptionsItemSelected()方法，在菜单的switch中添加：

```
 case R.id.menu_color:
        changeColor();
        break;
```

在NoteEditor中添加函数changeColor()：

```
 private final void changeColor() {
        Intent intent = new Intent(null,mUri);
        intent.setClass(NoteEditor.this,NoteColor.class);
        NoteEditor.this.startActivity(intent);
    }
```

新建布局note_color.xml

```
?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="horizontal" android:layout_width="match_parent"
    android:layout_height="match_parent">
    <ImageButton
        android:id="@+id/color_white"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorWhite"
        android:onClick="white"/>
    <ImageButton
        android:id="@+id/color_yellow"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorYellow"
        android:onClick="yellow"/>
    <ImageButton
        android:id="@+id/color_blue"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorBlue"
        android:onClick="blue"/>
    <ImageButton
        android:id="@+id/color_green"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorGreen"
        android:onClick="green"/>
    <ImageButton
        android:id="@+id/color_red"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorRed"
        android:onClick="red"/>
</LinearLayout>
```

新建NoteColor.java

```
public class NoteColor extends Activity {
    private Cursor mCursor;
    private Uri mUri;
    private int color;
    private static final int COLUMN_INDEX_TITLE = 1;
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_BACK_COLOR,
    };
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.note_color);
        //从NoteEditor传入的uri
        mUri = getIntent().getData();
        mCursor = managedQuery(
                mUri,        // The URI for the note that is to be retrieved.
                PROJECTION,  // The columns to retrieve
                null,        // No selection criteria are used, so no where columns are needed.
                null,        // No where columns are used, so no where values are needed.
                null         // No sort order is needed.
        );
    }
    @Override
    protected void onResume(){
        //执行顺序在onCreate之后
        if (mCursor != null) {
            mCursor.moveToFirst();
            color = mCursor.getInt(COLUMN_INDEX_TITLE);
        }
        super.onResume();
    }
    @Override
    protected void onPause() {
        //执行顺序在finish()之后，将选择的颜色存入数据库
        super.onPause();
        ContentValues values = new ContentValues();
        values.put(NotePad.Notes.COLUMN_NAME_BACK_COLOR, color);
        getContentResolver().update(mUri, values, null, null);
    }
    public void white(View view){
        color = NotePad.Notes.DEFAULT_COLOR;
        finish();
    }
    public void yellow(View view){
        color = NotePad.Notes.YELLOW_COLOR;
        finish();
    }
    public void blue(View view){
        color = NotePad.Notes.BLUE_COLOR;
        finish();
    }
    public void green(View view){
        color = NotePad.Notes.GREEN_COLOR;
        finish();
    }
    public void red(View view){
        color = NotePad.Notes.RED_COLOR;
        finish();
    }

}
```

在注册表AndroidManifest.xml中定义

```
<activity android:name="NoteColor"
    android:theme="@android:style/Theme.Holo.Light.Dialog"
    android:label="ChangeColor"
    android:windowSoftInputMode="stateVisible"/>
```





3、笔记排序功能，可以进行颜色排序，创建时间排序，修改时间排序

效果图：

图11 12 13

在文件list_options_menu.xml中添加

```
<item
    android:id="@+id/menu_sort"
    android:title="@string/menu_sort"
    android:icon="@android:drawable/ic_menu_sort_by_size"
    android:showAsAction="always" >
    <menu>
        <item
            android:id="@+id/menu_sort1"
            android:title="@string/menu_sort1"/>
        <item
            android:id="@+id/menu_sort2"
            android:title="@string/menu_sort2"/>
        <item
            android:id="@+id/menu_sort3"
            android:title="@string/menu_sort3"/>
        </menu>
    </item>
```

在NoteList菜单switch下添加case：

```
case R.id.menu_sort1:
        cursor = managedQuery(
                getIntent().getData(),            
                PROJECTION,                      
                null,                          
                null,                          
                NotePad.Notes._ID 
                );
        adapter = new MyCursorAdapter(
                this,
                R.layout.noteslist_item,
                cursor,
                dataColumns,
                viewIDs
        );
        setListAdapter(adapter);
        return true;
 //修改时间排序
    case R.id.menu_sort2:
        cursor = managedQuery(
                getIntent().getData(),          
                PROJECTION,                      
                null,                            
                null,                       
                NotePad.Notes.DEFAULT_SORT_ORDER 
        );
        adapter = new MyCursorAdapter(
                this,
                R.layout.noteslist_item,
                cursor,
                dataColumns,
                viewIDs
        );
        setListAdapter(adapter);
        return true;
    //颜色排序
    case R.id.menu_sort3:
        cursor = managedQuery(
                getIntent().getData(),
                PROJECTION,      
                null,       
                null,       
                NotePad.Notes.COLUMN_NAME_BACK_COLOR
                );
        adapter = new MyCursorAdapter(
                this,
                R.layout.noteslist_item,
                cursor,
                dataColumns,
                viewIDs
                );
        setListAdapter(adapter);
        return true;
```

将adapter,cursor,dataColumns,viewIDs定义在函数外类内

```
private MyCursorAdapter adapter;
private Cursor cursor;
private String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,  NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE } ;
private int[] viewIDs = { android.R.id.text1 , R.id.text1_time };
```

