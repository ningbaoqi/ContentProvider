### 创建ContentProvider
#### AndroidManifest.xml
```
<!--android:name 指定该ContentProvider的实现类的类名 android:exported指定该ContentProvider是否允许其他应用程序调用，如果该属性为true则允许其他应用程序调用，如果该属性为false，则该ContentProvider将不允许其他应用程序调用-->
<!--android:authorities 指定该android:authorities对应的Uri(相当于该ContentProvider分配一个域名)-->
<provider
    android:name=".MyContentProvider"
    android:authorities="ningbaoqi.com.contentprovider"
    android:exported="true"/>
```
#### MyContentProvider
```
 /* ContentProvider是不同应用程序之间进行数据交换的标准API，ContentProvider是以某种Uri的形式对外提供数据，允许其他应用程序访问或修改数据，其他应用程序使用ContentResolver根据Uri去访问操作指定数据
 * 一旦ContentProvider暴露了自己的数据操作接口，那么不管应用程序是否启动，其他应用程序都可以通过该接口来操作该应用程序的内部数据，包括增，删改，查等
 */

public class MyContentProvider extends ContentProvider {
    private MyDatabaseHelper databaseHelper;
    private SQLiteDatabase database;

    /**
     * 创建Uri匹配器对象，检测其他用户传入的uri与匹配器定义好的uri哪条匹配
     */
    private static UriMatcher uriMatcher = new UriMatcher(UriMatcher.NO_MATCH);//调用方的uri匹配不匹配时返回NO_MATCH即-1

    /**
     * 向匹配器中添加可以匹配的Uri
     * */
    static {
        uriMatcher.addURI("ningbaoqi.com.contentprovider", "person", 1);//在调用方的Uri需要是content://ningbaoqi.com.contentprovider/person才能匹配，返回码是1
        uriMatcher.addURI("ningbaoqi.com.contentprovider", "teacher", 2);//在调用方的Uri需要是content://ningbaoqi.com.contentprovider/teacher才能匹配，返回码是2
        uriMatcher.addURI("ningbaoqi.com.contentprovider", "person/#", 3);//在调用方的Uri需要是content://ningbaoqi.com.contentprovider/person/数字，才能匹配，返回码是3，其中数字这个的数据可以任意使用，一般作为ID使用
    }

    /**
     * 内容提供者创建时调用，当其他应用程序第一次访问ContentProvider时，该ContentProvider会被创建出来，并立即回调该方法
     */
    @Override
    public boolean onCreate() {
        databaseHelper = new MyDatabaseHelper(getContext(), "database.db", null, 1);
        database = databaseHelper.getWritableDatabase();
        return false;
    }

    /**
     * 根据该Uri查询出selection条件所匹配的全部记录，其中projection就是一个列名列表，表明只选择出指定的数据列，返回查询的Cursor
     */
    @Nullable
    @Override
    public Cursor query(@NonNull Uri uri, @Nullable String[] projection, @Nullable String selection, @Nullable String[] selectionArgs, @Nullable String sortOrder) {
        Cursor cursor = null;
        if (uriMatcher.match(uri) == 1) {
            cursor = database.query("person", projection, selection, selectionArgs, null, null, sortOrder, null);
        } else if (uriMatcher.match(uri) == 2) {
            cursor = database.query("teacher", projection, selection, selectionArgs, null, null, sortOrder, null);
        } else if (uriMatcher.match(uri) == 3) {
            /**
             * 把Uri末尾携带的数字提取出来
             * */
            long i = ContentUris.parseId(uri);
            cursor = database.query("person", projection, "_id = ? ", new String[]{i + ""}, null, null, sortOrder, null);
        } else {
            throw new IllegalArgumentException("Uri有问题");
        }
        return cursor;
    }

    /**
     * 该方法用于返回当前Uri所代表的数据的MIME类型，如果该Uri对应的数据可能包括多条记录，那么MIME类型字符串应该以 vnd.android.cursor.dir/开头,如果该Uri对应的数据只包含一条记录，那么MIME类型字符串应该以vnd.android.cursor.item/开头
     *  该方法没什么用，留做公司内部定义
     */
    @Nullable
    @Override
    public String getType(@NonNull Uri uri) {
        if (uriMatcher.match(uri) == 1) {
            return "vnd.android.cursor.dir/" + "person";
        } else if (uriMatcher.match(uri) == 3) {
            return "vnd.android.cursor.item/" + "person";
        }
        return null;
    }

    /**
     * 根据该Uri插入values对应的数据，该方法应该返回新插入的记录的Uri
     */
    @Nullable
    @Override
    public Uri insert(@NonNull Uri uri, @Nullable ContentValues values) {
        /**
         * 判断传入的uri是否和提供的匹配，
         * */
        if (uriMatcher.match(uri) == 1) {
            database.insert("person", null, values);
            /**
             * 发送数据变化的通知，所有注册在这个Uri上的内容观察者都可以收到这个通知
             * */
            getContext().getContentResolver().notifyChange(uri , null);
        } else if (uriMatcher.match(uri) == 2) {
            database.insert("teacher", null, values);
        } else {
            throw new IllegalArgumentException("uri有问题");
        }
       return uri;
    }

    /**
     * 根据该Uri删除selection条件匹配的全部记录，返回的是被删除的记录数
     */
    @Override
    public int delete(@NonNull Uri uri, @Nullable String selection, @Nullable String[] selectionArgs) {
        int number = database.delete("person", selection, selectionArgs);
        return number;
    }

    /**
     * 根据Uri修改selection条件所匹配的全部记录，返回被更新的记录的条数
     */
    @Override
    public int update(@NonNull Uri uri, @Nullable ContentValues values, @Nullable String selection, @Nullable String[] selectionArgs) {
        int number = database.update("person", values, selection, selectionArgs);
        return number;
    }
}
```
#### MyDatabaseHelper
```
public class MyDatabaseHelper extends SQLiteOpenHelper {

    public MyDatabaseHelper(Context context, String name, SQLiteDatabase.CursorFactory factory, int version) {
        super(context, name, factory, version);
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        db.execSQL("create table person(_id integer primary key autoincrement , name char(10) , money integer(20))");
        db.execSQL("create table teacher(_id integer primary key autoincrement , name char(10) , money integer(20))");
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {

    }
}
```
