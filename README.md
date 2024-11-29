## 基于NotePad应用的功能扩展

### 1. 原应用展示
图1：NotePad主界面

![image](https://github.com/user-attachments/assets/329f5597-e730-4699-a4e8-217da8ff8372)

图2：新建笔记

![image](https://github.com/user-attachments/assets/f639a733-d6a3-4a06-a927-f1e8d28ed064)

图3：新建笔记退回主页面

![image](https://github.com/user-attachments/assets/3f6e414b-2268-4707-a064-203b16d61886)

图4：进入笔记，编辑标题菜单

![image](https://github.com/user-attachments/assets/8716cc6a-1a71-4864-825d-2cd566bcbb3e)

图5：编辑标题

![image](https://github.com/user-attachments/assets/a684778f-c4c3-4222-a0d9-6b8a156c4ce5)

图6：笔记列表

![image](https://github.com/user-attachments/assets/1ef81f46-15f6-47db-80ae-bb82d2ab86f4)

### 2. 拓展功能

* NotesList中显示条目增加时间显示  
* 笔记查询（按标题查询）  
* UI美化（暗黑\明亮主题的转换）  
* 批量删除  
* 笔记收藏
* 添加图片
* 导出笔记  

### 3. 拓展应用码

源码：[NotePad](https://github.com/LYoxo/NotePad_new-master/tree/master)

扩展后的目录结构：

<img width="200" alt="11c3de87643bcc9680827daefef28d8" src="https://github.com/user-attachments/assets/346b94e6-1950-4fea-8e3d-11d57a1d62e2">

<img width="200" alt="b0aadc37eeac44d2feb3481168550ba" src="https://github.com/user-attachments/assets/45a2ee45-3ac5-45db-8daa-b85597fa8cba">


### 4. 拓展功能解析

#### 4.1 NotesList中显示条目增加时间显示
在NotePad原应用中，笔记列表只显示了笔记的标题。如图3、图6。要对它做时间扩展，可以把时间放在标题的下方。

1.找到notelist_item.xml布局文件中添加TextView 

（添加新的TextView时应添加垂直约束——android:gravity="center_vertical"）
```
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content">
    >
    <TextView xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@android:id/text1"
        android:layout_width="match_parent"
        android:layout_height="?android:attr/listPreferredItemHeight"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:gravity="center_vertical"
        android:paddingLeft="5dip"
        android:singleLine="true"
        />

    <TextView
        android:id="@android:id/text2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:gravity="center_vertical"
        android:paddingLeft="5dp"
        android:singleLine="true" />
```

2.在NoteList类的PROJECTION中添加COLUMN_NAME_MODIFICATION_DATE字段
```
private static final String[] PROJECTION = new String[] {
        NotePad.Notes._ID, // 0
        NotePad.Notes.COLUMN_NAME_TITLE, // 1
        NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,//在这里加入了修改时间的显示
};
```

3.修改适配器内容，在NoteList类增加dataColumns中装配到ListView的内容，所以要同时增加一个ID标识来存放该时间参数
```
// The names of the cursor columns to display in the view, initialized to the title column
String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,
        NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE  } ;

// The view IDs that will display the cursor columns, initialized to the TextView in
// noteslist_item.xml
int[] viewIDs = { android.R.id.text1 ,android.R.id.text2};
```

4.在NoteEditor类的updateNote方法中获取当前系统的时间，并对时间进行格式化
```
// Sets up a map to contain values to be updated in the provider.
        ContentValues values = new ContentValues();
        Long now = Long.valueOf(System.currentTimeMillis());
        SimpleDateFormat sf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        Date d = new Date(now);
        String format = sf.format(d);
        values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, format);
```
5.时间戳运行效果展示：

<img width="227" alt="8633313679f2a1e3e1e0efee5193851" src="https://github.com/user-attachments/assets/bb4bf0a7-bd5a-4f16-a6de-0d9f3b238638">

#### 4.2 笔记查询（按标题查询）
1.搜索组件在主页面的菜单选项中，因此在res—menu—list_options_menu.xml布局文件中添加搜索功能，新增menu_search
```
<item
    android:id="@+id/menu_search"
    android:icon="@android:drawable/ic_menu_search"
    android:title="@string/menu_search"
    android:showAsAction="always" />
```
2.在res—layout中新建一个查找笔记内容的布局文件note_search.xml
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
        />
    <ListView
        android:id="@+id/list_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        />
</LinearLayout>

```
3.在NoteList类中的onOptionsItemSelected方法中新增search查询的处理(跳转)
```
case R.id.menu_search:
    //查找功能
    //startActivity(new Intent(Intent.ACTION_SEARCH, getIntent().getData()));
    Intent intent = new Intent(this, NoteSearch.class);
    this.startActivity(intent);
    return true;
```
4.新建一个NoteSearch类用于search功能的功能实现

PS:注意在res—values—strings.xml中添加menu_search字段
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
        setContentView(R.layout.note_search);
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
5.搜索框效果展示

<img width="236" alt="413481b01bb697146e37e8d1829b130" src="https://github.com/user-attachments/assets/cdbd2a55-51b3-423b-adce-1aee04eaf28f">

#### 4.3 UI美化（暗黑\明亮主题的转换） 
1.首先在 list_options_menu.xml 中添加新的菜单项：

PS:在 strings.xml 中添加新的字段
```
<item
        android:id="@+id/menu_theme"
        android:title="@string/menu_theme"
        android:showAsAction="never" />
```
2.修改 NotesList.java 中的 onOptionsItemSelected 方法：
```
@Override
public boolean onOptionsItemSelected(MenuItem item) {
    switch (item.getItemId()) {
        case R.id.menu_theme:
             // 获取当前主题设置
                SharedPreferences settings = getSharedPreferences(PREFS_NAME, Context.MODE_PRIVATE);
                boolean isDarkTheme = settings.getBoolean(THEME_KEY, false);
                
                // 切换主题并保存设置
                SharedPreferences.Editor editor = settings.edit();
                editor.putBoolean(THEME_KEY, !isDarkTheme);
                editor.apply();
                
                // 重新创建活动以应用新主题
                recreate();
                return true;
        // ... 其他 case 保持不变
    }
}
```
3.在 AndroidManifest.xml 中为 NotesList 活动添加主题属性：
```
<activity android:name=".NotesList"
          android:theme="@android:style/Theme.Holo.Light"
          android:label="@string/title_notes_list">
</activity>
```
4.为了保存主题设置以便在应用重启时保持选择的主题,在 NotesList.java 中添加主题设置的保存和读取
```
public class NotesList extends ListActivity {
    private static final String PREFS_NAME = "NotepadPrefs";
    private static final String THEME_KEY = "theme_key";
 @Override
    protected void onCreate(Bundle savedInstanceState) {
        // 在 super.onCreate() 之前设置主题
        SharedPreferences settings = getSharedPreferences(PREFS_NAME, Context.MODE_PRIVATE);
        boolean isDarkTheme = settings.getBoolean(THEME_KEY, false);
        setTheme(isDarkTheme ? android.R.style.Theme_Holo : android.R.style.Theme_Holo_Light);
        
        super.onCreate(savedInstanceState);
        // ... 其余代码保持不变
    }

```
5.UI美化效果展示：

变化前：

<img width="237" alt="0f1b9a74382b14566e0eb479faaf6a9" src="https://github.com/user-attachments/assets/729395d8-63f9-4f5d-8106-01166eb786ac">

变化后：

<img width="234" alt="b897c63d70e008342f0d94516783786" src="https://github.com/user-attachments/assets/309a9374-f9b0-400f-bc14-4fee23e8648e">

#### 4.4 批量删除  
1.首先添加成员变量来跟踪批量删除模式：
```
private boolean isInBatchDeleteMode = false;
private Set<Long> selectedNotes = new HashSet<>();
```
2.添加切换批量删除模式的方法：
```
private void toggleBatchDeleteMode() {
        isInBatchDeleteMode = !isInBatchDeleteMode;
        MenuItem batchDeleteItem = mMenu.findItem(R.id.menu_batch_delete);
        batchDeleteItem.setVisible(isInBatchDeleteMode);

        if (!isInBatchDeleteMode) {
            selectedNotes.clear();
        }

        invalidateOptionsMenu();
        getListView().invalidateViews();
    }
```
3.在NoteList.java中，修改 SimpleCursorAdapter 的 bindView 方法：
```
 //批量删除
CheckBox deleteCheckBox = (CheckBox) view.findViewById(R.id.delete_checkbox);
CheckBox selectCheckBox = (CheckBox) view.findViewById(R.id.select_checkbox);
 int todoStatus = cursor.getInt(cursor.getColumnIndexOrThrow(NotePad.Notes.COLUMN_NAME_TODO_STATUS));
    todoCheckBox.setChecked(todoStatus == 1);
 // 处理批量删除模式
final long noteId = cursor.getLong(cursor.getColumnIndexOrThrow(NotePad.Notes._ID));
deleteCheckBox.setVisibility(isInBatchDeleteMode ? View.VISIBLE : View.GONE);
deleteCheckBox.setChecked(selectedNotes.contains(noteId));
deleteCheckBox.setOnCheckedChangeListener((buttonView, isChecked) -> {
                    if (isChecked) {
                        selectedNotes.add(noteId);
                    } else {
                        selectedNotes.remove(noteId);
                    }
                });
```
4.在菜单项中添加删除选中项的操作：
```
 case R.id.menu_batch_delete:
                if (selectedNotes.isEmpty()) {
                    Toast.makeText(this, "请先选择要删除的笔记", Toast.LENGTH_SHORT).show();
                } else {
                    StringBuilder selection = new StringBuilder();
                    selection.append(NotePad.Notes._ID).append(" IN (");
                    String[] selectionArgs = new String[selectedNotes.size()];

                    int i = 0;
                    for (Long noteId : selectedNotes) {
                        if (i > 0) {
                            selection.append(",");
                        }
                        selection.append("?");
                        selectionArgs[i] = String.valueOf(noteId);
                        i++;
                    }
                    selection.append(")");

                    getContentResolver().delete(
                            NotePad.Notes.CONTENT_URI,
                            selection.toString(),
                            selectionArgs
                    );

                    selectedNotes.clear();
                    Toast.makeText(this, "已删除选中的笔记", Toast.LENGTH_SHORT).show();
                }
                toggleBatchDeleteMode();
                return true;
```
5.在布局文件 noteslist_item.xml 中添加用于批量删除的 CheckBox：
```
<CheckBox
    android:id="@+id/delete_checkbox"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:visibility="gone" />
```
6.实现长按批量选择，在 onCreate 方法中添加长按监听器：
```
 ListView listView = getListView();
    listView.setOnItemLongClickListener((parent, view, position, id) -> {
        if (!isInBatchDeleteMode) {
            toggleBatchDeleteMode();
            MenuItem batchDeleteItem = mMenu.findItem(R.id.menu_batch_delete);
            batchDeleteItem.setVisible(true);
            
            // 选中长按的项
            selectedNotes.add(id);
            view.findViewById(R.id.delete_checkbox).setVisibility(View.VISIBLE);
            ((CheckBox) view.findViewById(R.id.delete_checkbox)).setChecked(true);
        }
        return true;
    });
```
7.修改 SimpleCursorAdapter 的创建和绑定：
```
SimpleCursorAdapter adapter = new SimpleCursorAdapter(
    this,
    R.layout.noteslist_item,
    cursor,
    new String[] {
        NotePad.Notes.COLUMN_NAME_TITLE,
        NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,
        NotePad.Notes.COLUMN_NAME_TODO_STATUS
    },
    new int[] {
        android.R.id.text1,
        android.R.id.text2,
        R.id.todo_checkbox
    }
) {
    @Override
    public void bindView(View view, Context context, Cursor cursor) {
        super.bindView(view, context, cursor);
        
        CheckBox deleteCheckBox = view.findViewById(R.id.delete_checkbox);
        long noteId = cursor.getLong(cursor.getColumnIndex(NotePad.Notes._ID));
        
        deleteCheckBox.setVisibility(isInBatchDeleteMode ? View.VISIBLE : View.GONE);
        deleteCheckBox.setChecked(selectedNotes.contains(noteId));
        
        deleteCheckBox.setOnCheckedChangeListener((buttonView, isChecked) -> {
            if (isChecked) {
                selectedNotes.add(noteId);
            } else {
                selectedNotes.remove(noteId);
            }
        });
    }
};
```
8.批量删除的结果展示：

选择：

<img width="278" alt="4a89a3ae8b5a8b5c6969ac3f53493b3" src="https://github.com/user-attachments/assets/ca663c11-fc15-430f-a958-934341fb1111">

删除后：

<img width="273" alt="279127e0d9fc4ddc9488c9241ba52b8" src="https://github.com/user-attachments/assets/5d4ddf77-c822-4784-adb4-aae2e5e675b0">

#### 4.5 笔记收藏
1.首先在 NotePad.java 中添加收藏状态列:
```
public static final String COLUMN_NAME_FAVORITE = "favorite";
```
2.修改 NotePadProvider.java 中的数据库创建语句:
```
static class DatabaseHelper extends SQLiteOpenHelper {
    private static final String DATABASE_NAME = "notes.db";
    private static final int DATABASE_VERSION = 3;  // 增加版本号

    private static final String DATABASE_CREATE =
        "CREATE TABLE " + NotePad.Notes.TABLE_NAME + " ("
            + NotePad.Notes._ID + " INTEGER PRIMARY KEY,"
            + NotePad.Notes.COLUMN_NAME_TITLE + " TEXT,"
            + NotePad.Notes.COLUMN_NAME_NOTE + " TEXT,"
            + NotePad.Notes.COLUMN_NAME_CREATE_DATE + " INTEGER,"
            + NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + " INTEGER,"
            + NotePad.Notes.COLUMN_NAME_TODO_STATUS + " INTEGER DEFAULT 0,"
            + NotePad.Notes.COLUMN_NAME_FAVORITE + " INTEGER DEFAULT 0"
            + ");";

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
         if (oldVersion < 3) {
            try {
                db.execSQL("ALTER TABLE " + NotePad.Notes.TABLE_NAME + 
                    " ADD COLUMN " + NotePad.Notes.COLUMN_NAME_FAVORITE + 
                    " INTEGER DEFAULT 0");
            } catch (Exception e) {
                // 列可能已经存在，忽略错误
            }
        }
    }
}
```
3.在 list_options_menu.xml 中添加收藏菜单项:
```
<item
    android:id="@+id/menu_favorites"
    android:icon="@android:drawable/btn_star"
    android:title="收藏夹"
    android:showAsAction="always" />
```
4.创建新的 FavoritesList.java Activity:
```
public class FavoritesList extends ListActivity {
    private static final String[] PROJECTION = new String[] {
        NotePad.Notes._ID,
        NotePad.Notes.COLUMN_NAME_TITLE,
        NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,
        NotePad.Notes.COLUMN_NAME_FAVORITE
    };
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        
        String selection = NotePad.Notes.COLUMN_NAME_FAVORITE + "=1";
        Cursor cursor = managedQuery(
            NotePad.Notes.CONTENT_URI,
            PROJECTION,
            selection,
            null,
            NotePad.Notes.DEFAULT_SORT_ORDER
        );
        
        SimpleCursorAdapter adapter = new SimpleCursorAdapter(
            this,
            R.layout.noteslist_item,
            cursor,
            new String[] { NotePad.Notes.COLUMN_NAME_TITLE },
            new int[] { android.R.id.text1 }
        );
        setListAdapter(adapter);
    }
}
```
5.在 NotesList.java 的 onOptionsItemSelected 方法中添加处理:
```
case R.id.menu_favorites:
    Intent intent = new Intent(this, FavoritesList.class);
    favoritesIntent.setData(getIntent().getData());
    startActivity(intent);
    return true;
```
6.在 AndroidManifest.xml 中注册新 Activity:
```
<activity
    android:name=".FavoritesList"
    android:label="收藏夹"
    android:exported="true" />
```
7.修改 NotesList.java 中的上下文菜单,添加收藏功能:
```
case R.id.context_favorite:
    ContentValues values = new ContentValues();
    values.put(NotePad.Notes.COLUMN_NAME_FAVORITE, 1);
    getContentResolver().update(noteUri, values, null, null);
    Toast.makeText(this, "已添加到收藏夹", Toast.LENGTH_SHORT).show();
    return true;
```
8.修改 NotesList.java 中的 PROJECTION：
```
private static final String[] PROJECTION = new String[] {
    NotePad.Notes._ID,
    NotePad.Notes.COLUMN_NAME_TITLE,
    NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,
    NotePad.Notes.COLUMN_NAME_TODO_STATUS,
    NotePad.Notes.COLUMN_NAME_FAVORITE
};
```
9.在 NotePadProvider.java 中添加收藏列的映射：
```
static {
    sNotesProjectionMap = new HashMap<String, String>();
    sNotesProjectionMap.put(NotePad.Notes._ID, NotePad.Notes._ID);
    sNotesProjectionMap.put(NotePad.Notes.COLUMN_NAME_TITLE, NotePad.Notes.COLUMN_NAME_TITLE);
    sNotesProjectionMap.put(NotePad.Notes.COLUMN_NAME_NOTE, NotePad.Notes.COLUMN_NAME_NOTE);
    sNotesProjectionMap.put(NotePad.Notes.COLUMN_NAME_CREATE_DATE, NotePad.Notes.COLUMN_NAME_CREATE_DATE);
    sNotesProjectionMap.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE);
    sNotesProjectionMap.put(NotePad.Notes.COLUMN_NAME_TODO_STATUS, NotePad.Notes.COLUMN_NAME_TODO_STATUS);
    sNotesProjectionMap.put(NotePad.Notes.COLUMN_NAME_FAVORITE, NotePad.Notes.COLUMN_NAME_FAVORITE);
}
```
10.在 NotesList.java 中添加批量收藏相关的变量和方法：
```
private boolean isInBatchMode = false;
private Set<Long> selectedNotes = new HashSet<>();
private Menu mMenu;

 private void toggleBatchMode() {
        isInBatchMode = !isInBatchMode;
        MenuItem batchDeleteItem = mMenu.findItem(R.id.menu_batch_delete);
        MenuItem batchFavoriteItem = mMenu.findItem(R.id.menu_favorites);

        batchDeleteItem.setVisible(isInBatchMode);
        batchFavoriteItem.setIcon(isInBatchMode ? android.R.drawable.ic_menu_add : android.R.drawable.btn_star);
        batchFavoriteItem.setTitle(isInBatchMode ? "添加到收藏" : "收藏夹");

        if (!isInBatchMode) {
            selectedNotes.clear();
        }
        invalidateOptionsMenu();
        getListView().invalidateViews();
    }
```
11.修改 onOptionsItemSelected 中的收藏处理：
```
case R.id.menu_favorites:
                if (isInBatchMode) {
                    if (selectedNotes.isEmpty()) {
                        Toast.makeText(this, "请先选择要收藏的笔记", Toast.LENGTH_SHORT).show();
                    } else {
                        StringBuilder selection = new StringBuilder();
                        selection.append(NotePad.Notes._ID).append(" IN (");
                        String[] selectionArgs = new String[selectedNotes.size()];

                        int i = 0;
                        for (Long noteId : selectedNotes) {
                            if (i > 0) {
                                selection.append(",");
                            }
                            selection.append("?");
                            selectionArgs[i] = String.valueOf(noteId);
                            i++;
                        }
                        selection.append(")");

                        ContentValues values = new ContentValues();
                        values.put(NotePad.Notes.COLUMN_NAME_FAVORITE, 1);
                        getContentResolver().update(
                                NotePad.Notes.CONTENT_URI,
                                values,
                                selection.toString(),
                                selectionArgs
                        );

                        Toast.makeText(this, "已添加到收藏夹", Toast.LENGTH_SHORT).show();
                        selectedNotes.clear();
                    }
                    toggleBatchMode();
                } else {
                    Intent favoritesIntent = new Intent(this, FavoritesList.class);
                    favoritesIntent.setData(getIntent().getData());
                    startActivity(favoritesIntent);
                }
                return true;
```
12.修改 SimpleCursorAdapter 的 bindView 方法:
```
@Override
public void bindView(View view, Context context, Cursor cursor) {
    super.bindView(view, context, cursor);
    
    CheckBox selectCheckBox = view.findViewById(R.id.select_checkbox);
    long noteId = cursor.getLong(cursor.getColumnIndex(NotePad.Notes._ID));
    
    selectCheckBox.setVisibility(isInBatchMode ? View.VISIBLE : View.GONE);
    selectCheckBox.setChecked(selectedNotes.contains(noteId));
    
    selectCheckBox.setOnCheckedChangeListener((buttonView, isChecked) -> {
        if (isChecked) {
            selectedNotes.add(noteId);
        } else {
            selectedNotes.remove(noteId);
        }
    });
}
```
13.在实现收藏夹的基础上，实现在收藏夹里，可以继续点击进行修改，需要在FavoritesList.java 中，添加点击事件处理。在 onCreate 方法中添加以下代码：
```
@Override
protected void onListItemClick(ListView l, View v, int position, long id) {
    // 构建笔记的 URI
    Uri noteUri = ContentUris.withAppendedId(getIntent().getData(), id);
    // 启动编辑活动
    startActivity(new Intent(Intent.ACTION_EDIT, noteUri));
}
```
14.笔记收藏的结果展示：

<img width="306" alt="fab834031b668a5e004252dea54e52a" src="https://github.com/user-attachments/assets/dcfdbd96-e786-47aa-ae3c-9be1209916eb">

收藏夹：

<img width="295" alt="e389198d7709ac9a66f36707460e28b" src="https://github.com/user-attachments/assets/c1f09755-7c07-4671-8286-9fe9dad491fa">

#### 4.6 添加图片
1.首先修改数据库结构，在 NotePadProvider.java 中添加图片路径列：
```
public static final String COLUMN_NAME_IMAGE_PATH = "image_path";

private static final String DATABASE_CREATE =
    "CREATE TABLE " + NotePad.Notes.TABLE_NAME + " ("
    + NotePad.Notes._ID + " INTEGER PRIMARY KEY,"
    + NotePad.Notes.COLUMN_NAME_TITLE + " TEXT,"
    + NotePad.Notes.COLUMN_NAME_NOTE + " TEXT,"
    + NotePad.Notes.COLUMN_NAME_CREATE_DATE + " INTEGER,"
    + NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + " INTEGER,"
    + NotePad.Notes.COLUMN_NAME_TODO_STATUS + " INTEGER DEFAULT 0,"
    + NotePad.Notes.COLUMN_NAME_FAVORITE + " INTEGER DEFAULT 0,"
    + NotePad.Notes.COLUMN_NAME_IMAGE_PATH + " TEXT"
    + ");";
```
2.在 NoteEditor.java 中添加选择图片的功能：
```
private static final int REQUEST_CODE_SELECT_IMAGE = 2;
private ImageView noteImage;
private String currentImagePath;

private void selectImage() {
    Intent intent = new Intent(Intent.ACTION_PICK);
    intent.setType("image/*");
    startActivityForResult(intent, REQUEST_CODE_SELECT_IMAGE);
}

@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    if (requestCode == REQUEST_CODE_SELECT_IMAGE && resultCode == RESULT_OK) {
        Uri selectedImage = data.getData();
        String[] filePathColumn = {MediaStore.Images.Media.DATA};
        
        Cursor cursor = getContentResolver().query(selectedImage,
                filePathColumn, null, null, null);
        cursor.moveToFirst();
        
        int columnIndex = cursor.getColumnIndex(filePathColumn[0]);
        currentImagePath = cursor.getString(columnIndex);
        cursor.close();
        
        // 显示选择的图片
        noteImage.setImageBitmap(BitmapFactory.decodeFile(currentImagePath));
        noteImage.setVisibility(View.VISIBLE);
    }
}
```
3.修改笔记编辑布局文件 note_editor.xml：
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <EditText
        android:id="@+id/note"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:gravity="top" />
        
    <ImageView
        android:id="@+id/note_image"
        android:layout_width="match_parent"
        android:layout_height="200dp"
        android:scaleType="centerCrop"
        android:visibility="gone" />
        
    <Button
        android:id="@+id/add_image"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="添加图片" />
</LinearLayout>
```
4.在保存笔记时，保存图片路径：
```
private final void updateNote(String text, String title) {
    ContentValues values = new ContentValues();
    values.put(NotePad.Notes.COLUMN_NAME_NOTE, text);
    values.put(NotePad.Notes.COLUMN_NAME_TITLE, title);
    values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, System.currentTimeMillis());
    if (currentImagePath != null) {
        values.put(NotePad.Notes.COLUMN_NAME_IMAGE_PATH, currentImagePath);
    }
    
    getContentResolver().update(
        mUri,
        values,
        null,
        null
    );
}
```
5.添加图片的结果展示：

PS:由于虚拟机中没有图片，所以这个功能具体能否实现有待考察

<img width="282" alt="f3efa62a63472d368ed1fb0e716cc0d" src="https://github.com/user-attachments/assets/b6ea76a4-31f5-43fc-8129-1f415f24ebf3">

点击添加照片按钮：

<img width="281" alt="d49211b4246ec24894bde51eccf4462" src="https://github.com/user-attachments/assets/0dda5cee-a27a-49b6-b49d-869540ba3358">


#### 4.7 导出笔记
1.先在菜单文件中添加一个导出笔记的选项，editor_options_menu.xml：
```
<item android:id="@+id/menu_output"
    android:title="@string/menu_output" />
```
2.在NoteEditor中找到onOptionsItemSelected()方法，在菜单的switch中添加：
```
//导出笔记选项
   case R.id.menu_output:
        outputNote();
        break;
```
3.在NoteEditor中添加函数outputNote()：
```
//跳转导出笔记的activity，将uri信息传到新的activity
    private final void outputNote() {
        Intent intent = new Intent(null,mUri);
        intent.setClass(NoteEditor.this,OutputText.class);
        NoteEditor.this.startActivity(intent);
    }
```
4.要对选择导出文件界面进行布局，新建布局output_text.xml，垂直线性布局放置EditText和Button:
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:paddingLeft="6dip"
    android:paddingRight="6dip"
    android:paddingBottom="3dip">
    <EditText android:id="@+id/output_name"
        android:maxLines="1"
        android:layout_marginTop="2dp"
        android:layout_marginBottom="15dp"
        android:layout_width="wrap_content"
        android:ems="25"
        android:layout_height="wrap_content"
        android:autoText="true"
        android:capitalize="sentences"
        android:scrollHorizontally="true" />
    <Button android:id="@+id/output_ok"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="right"
        android:text="@string/output_ok"
        android:onClick="OutputOk" />
</LinearLayout>
```
5.创建OutputText的Acitvity:
```
public class OutputText extends Activity {
   //要使用的数据库中笔记的信息
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_NOTE, // 2
            NotePad.Notes.COLUMN_NAME_CREATE_DATE, // 3
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 4
    };
    //读取出的值放入这些变量
    private String TITLE;
    private String NOTE;
    private String CREATE_DATE;
    private String MODIFICATION_DATE;
    //读取该笔记信息
    private Cursor mCursor;
    //导出文件的名字
    private EditText mName;
    //NoteEditor传入的uri，用于从数据库查出该笔记
    private Uri mUri;
    //关于返回与保存按钮的一个特殊标记，返回的话不执行导出，点击按钮才导出
    private boolean flag = false;
    private static final int COLUMN_INDEX_TITLE = 1;
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.output_text);
        mUri = getIntent().getData();
        mCursor = managedQuery(
                mUri,        // The URI for the note that is to be retrieved.
                PROJECTION,  // The columns to retrieve
                null,        // No selection criteria are used, so no where columns are needed.
                null,        // No where columns are used, so no where values are needed.
                null         // No sort order is needed.
        );
        mName = (EditText) findViewById(R.id.output_name);
    }
    @Override
    protected void onResume(){
        super.onResume();
        if (mCursor != null) {
            // The Cursor was just retrieved, so its index is set to one record *before* the first
            // record retrieved. This moves it to the first record.
            mCursor.moveToFirst();
            //编辑框默认的文件名为标题，可自行更改
            mName.setText(mCursor.getString(COLUMN_INDEX_TITLE));
        }
    }
    @Override
    protected void onPause() {
        super.onPause();
        if (mCursor != null) {
        //从mCursor读取对应值
            TITLE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_TITLE));
            NOTE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_NOTE));
            CREATE_DATE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_CREATE_DATE));
            MODIFICATION_DATE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE));
            //flag在点击导出按钮时会设置为true，执行写文件
            if (flag == true) {
                write();
            }
            flag = false;
        }
    }
    public void OutputOk(View v){
        flag = true;
        finish();
    }
    private void write()
    {
        try
        {
            // 如果手机插入了SD卡，而且应用程序具有访问SD的权限
            if (Environment.getExternalStorageState().equals(
                    Environment.MEDIA_MOUNTED)) {
                // 获取SD卡的目录
                File sdCardDir = Environment.getExternalStorageDirectory();
                //创建文件目录
                File targetFile = new File(sdCardDir.getCanonicalPath() + "/" + mName.getText() + ".txt");
                //写文件
                PrintWriter ps = new PrintWriter(new OutputStreamWriter(new FileOutputStream(targetFile), "UTF-8"));
                ps.println(TITLE);
                ps.println(NOTE);
                ps.println("创建时间：" + CREATE_DATE);
                ps.println("最后一次修改时间：" + MODIFICATION_DATE);
                ps.close();
                Toast.makeText(this, "保存成功,保存位置：" + sdCardDir.getCanonicalPath() + "/" + mName.getText() + ".txt", Toast.LENGTH_LONG).show();
            }
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
    }
}
```
6.在AndroidManifest.xml中将这个Acitvity主题定义为对话框样式，并且加入权限：
```
<!--添加导出activity-->
        <activity android:name="OutputText"
            android:label="@string/output_name"
            android:theme="@android:style/Theme.Holo.Dialog"
            android:windowSoftInputMode="stateVisible">
        </activity>
```
```
 <!-- 在SD卡中创建与删除文件权限 -->
    <uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS"/>
    <!-- 向SD卡写入数据权限 -->
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```
7.导出笔记的运行结果：

<img width="260" alt="7e4b5c0a4d91fce06d7f75dd0badfc4" src="https://github.com/user-attachments/assets/d6f1f4c7-f22e-4ae1-8278-630b287914eb">

系统文件夹中：

<img width="223" alt="f5865aff8ca07b50821d820c68af85b" src="https://github.com/user-attachments/assets/eb3e9417-0221-4cb4-931b-4055ba2c202b">
