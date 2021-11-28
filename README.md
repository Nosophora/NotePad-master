### 主要功能1：增加日期显示

#### 效果图：

![日期显示效果图]()

#### 实现方法：

在布局文件notelist_item.xml中增加一个TextView来显示日期
```
<TextView
android:id="@+id/text2"
android:layout_width="match_parent"
android:layout_height="match_parent"
android:textAppearance="?android:attr/textAppearanceLarge"
android:textSize="12dp"
android:gravity="center_vertical"
android:paddingLeft="10dip"
android:singleLine="true"
android:layout_weight="1"
android:layout_margin="0dp"
/>
```
在NodeEditor.java中,找到updateNode()函数，新增修改时间字段，并将其格式化存入数据库
```
Date nowTime = new Date(System.currentTimeMillis());
SimpleDateFormat sdFormatter = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
String retStrFormatNowDate = sdFormatter.format(nowTime);
// Sets up a map to contain values to be updated in the provider.
ContentValues values = new ContentValues();
values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, retStrFormatNowDate);
```
在NoteList.java的PROJECTION数组中增加该字段的描述，并在SimpleCursorAdapter中的参数viewsIDs和dataColumns增加子段描述，以达到将其读出和显示的目的
```
private static final String[] PROJECTION = new String[] {
NotePad.Notes._ID, // 0
NotePad.Notes.COLUMN_NAME_TITLE, // 1
NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,
};
    /** The index of the title column */
private static final int COLUMN_INDEX_TITLE = 1;
private SimpleCursorAdapter adapter;
    // The names of the cursor columns to display in the view, initialized to the title column
private String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE} ;
    // The view IDs that will display the cursor columns, initialized to the TextView in
    // noteslist_item.xml
private int[] viewIDs = { R.id.text1,R.id.text2 };
```
------

### 主要功能2：增加搜索显示

#### 效果图：

![搜索效果图]()

#### 实现方法：

在list_options_menu.xml中添加搜索item
```
<item
android:id="@+id/menu_search"
android:title="menu_search"
android:icon="@android:drawable/ic_search_category_default"
android:showAsAction="always">
</item>
```
在layout中新建布局文件note_search_list.xml
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
android:orientation="vertical" android:layout_width="match_parent"
android:layout_height="match_parent">
<SearchView
android:id="@+id/search_view"
android:layout_width="match_parent"
android:layout_height="wrap_content"
android:iconifiedByDefault="false"
android:queryHint="输入搜索内容..."
android:layout_alignParentTop="true">
</SearchView>
<ListView
android:id="@android:id/list"
android:layout_width="match_parent"
android:layout_height="wrap_content">
</ListView>
</LinearLayout>
```

创建NoteSearch.java,继承ListView,实现SearchView.OnQueryTextListener接口
```
package com.example.android.notepad;

import android.app.ListActivity;
import android.content.ContentUris;
import android.content.Intent;
import android.database.Cursor;
import android.net.Uri;
import android.os.Bundle;
import android.view.View;
import android.widget.ListView;
import android.widget.SearchView;
import android.widget.SimpleCursorAdapter;

public class NoteSearch extends ListActivity implements SearchView.OnQueryTextListener {
private static final String[] PROJECTION = new String[] {
NotePad.Notes._ID, // 0
NotePad.Notes.COLUMN_NAME_TITLE, // 1
//扩展 显示时间
NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 2

    };
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.note_search_list);
        Intent intent = getIntent();
        if (intent.getData() == null) {
            intent.setData(NotePad.Notes.CONTENT_URI);
        }
        SearchView searchview = (SearchView)findViewById(R.id.search_view);
        //为查询文本框注册监听器
        searchview.setOnQueryTextListener(NoteSearch.this);
    }
    @Override
    public boolean onQueryTextSubmit(String query) {
        return false;
    }
    @Override
    public boolean onQueryTextChange(String newText) {
        String selection = NotePad.Notes.COLUMN_NAME_TITLE + " Like ? ";
        String[] selectionArgs = { "%"+newText+"%" };
        Cursor cursor = managedQuery(
                getIntent().getData(),
                PROJECTION,
                selection,
                selectionArgs,
                NotePad.Notes.DEFAULT_SORT_ORDER
        );
        String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,  NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE };
        int[] viewIDs = { R.id.text1 , R.id.text2 };
        SimpleCursorAdapter adapter = new SimpleCursorAdapter(
                this,
                R.layout.noteslist_item,
                cursor,
                dataColumns,
                viewIDs
        );
        setListAdapter(adapter);
        return true;
    }
    @Override
    protected void onListItemClick(ListView l, View v, int position, long id) {

        Uri uri = ContentUris.withAppendedId(getIntent().getData(), id);
        String action = getIntent().getAction();
        if (Intent.ACTION_PICK.equals(action) || Intent.ACTION_GET_CONTENT.equals(action)) {
            setResult(RESULT_OK, new Intent().setData(uri));
        } else {
            startActivity(new Intent(Intent.ACTION_EDIT, uri));
        }
    }
}
```
在AndroidManifest.xml中注册NoteSearch
```
<activity
android:name="NoteSearch"
android:label="title_notes_search">
</activity>
```
------
