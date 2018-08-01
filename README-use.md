### 使用ContentProvider
#### MainActivity
```
public class MainActivity extends AppCompatActivity {

    private ContentResolver contentResolver;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        contentResolver = getContentResolver();
        /**
         * 注册一个内容观察者，监听短信数据库内容的变化，第二个参数如果是 true 那么只要以content://ningbaoqi.com.contentprovider/开头的Uri的数据变化，就能收到数据库变化的通知 ， 如果为false就需要精确匹配
         * */
        contentResolver.registerContentObserver(Uri.parse("content://ningbaoqi.com.contentprovider/"), true, new MyContentObserver(new Handler()));
    }

    public void insert(View view) {
        ContentValues values = new ContentValues();
        values.put("name", "zhangsan");
        values.put("money", "123");
        contentResolver.insert(Uri.parse("content://ningbaoqi.com.contentprovider/teacher"), values);
        values.clear();
        values.put("name", "lisi");
        values.put("money", "2321");
        contentResolver.insert(Uri.parse("content://ningbaoqi.com.contentprovider/person"), values);
        values.clear();
        values.put("name", "wangwu");
        values.put("money", "213123");
        contentResolver.insert(Uri.parse("content://ningbaoqi.com.contentprovider/teacher"), values);
    }

    public void delete(View view) {
        contentResolver.delete(Uri.parse("content://ningbaoqi.com.contentprovider"), "name = ?", new String[]{"wangwu"});
    }

    public void update(View view) {
        ContentValues values = new ContentValues();
        values.put("money", "11111");
        contentResolver.update(Uri.parse("content://ningbaoqi.com.contentprovider"), values, "name = ?", new String[]{"lisi"});
    }

    public void query(View view) {
        Cursor cursor = contentResolver.query(Uri.parse("content://ningbaoqi.com.contentprovider/person/7"), null, null, null, null);
        while (cursor.moveToNext()) {
            String name = cursor.getString(cursor.getColumnIndex("name"));
            int money = cursor.getInt(cursor.getColumnIndex("money"));
            Toast.makeText(this, "name : " + name + "---------" + "money : " + money, Toast.LENGTH_LONG).show();
        }
    }

    public void getType(View view) {
        String getType = contentResolver.getType(Uri.parse("content://ningbaoqi.com.contentprovider/person"));
        Toast.makeText(this, getType, Toast.LENGTH_LONG).show();
    }

    class MyContentObserver extends ContentObserver {

        /**
         * Creates a content observer.
         *
         * @param handler The handler to run {@link #onChange} on, or null if none.
         */
        public MyContentObserver(Handler handler) {
            super(handler);
        }

        /**
         * 收到数据变化的通知此方法调用
         * */
        @Override
        public void onChange(boolean selfChange) {
            super.onChange(selfChange);
            Toast.makeText(MainActivity.this, "监听的contentprovider变化了", Toast.LENGTH_LONG).show();
        }
    }
}
```
