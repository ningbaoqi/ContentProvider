### 使用ContentProvider查询和插入联系人
#### AndroidManifest.xml
```
<uses-permission android:name="android.permission.READ_CONTACTS"/>
<uses-permission android:name="android.permission.WRITE_CONTACTS"/>
```
#### Activity
```
public class MainActivity extends AppCompatActivity {

    private TextView textView;
    StringBuilder stringBuilder = new StringBuilder();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        textView = (TextView) findViewById(R.id.text);
        ContentResolver contentResolver = getContentResolver();
        Cursor cursor = contentResolver.query(Uri.parse("content://com.android.contacts/raw_contacts"), new String[]{"contact_id"}, null, null, null);
        while (cursor.moveToNext()) {
            String contactID = cursor.getString(0);
            Cursor dataCursor = contentResolver.query(Uri.parse("content://com.android.contacts/data"), new String[]{"data1", "mimetype"}, "raw_contact_id = ?", new String[]{contactID} , null);
            while (dataCursor.moveToNext()) {
                String data1 = dataCursor.getString(0);
                stringBuilder.append(data1 + "\n");
            }
            dataCursor.close();
        }
        cursor.close();
        textView.setText(stringBuilder.toString());
//        可以通过API来进行操作
//        ContentResolver contentResolver = getContentResolver();
//        Cursor cursor = contentResolver.query(ContactsContract.Contacts.CONTENT_URI, new String[]{ContactsContract.Contacts._ID, ContactsContract.Contacts.DISPLAY_NAME}, null, null, null);
//        if (cursor != null) {
//            while (cursor.moveToNext()) {
//                int id = cursor.getInt(cursor.getColumnIndex(ContactsContract.Contacts._ID));
//                String name = cursor.getString(cursor.getColumnIndex(ContactsContract.Contacts.DISPLAY_NAME));
//                stringBuilder.append("id:   " + id + "-------" + "name:   " + name + "\n");
//                Cursor inCur = contentResolver.query(ContactsContract.CommonDataKinds.Phone.CONTENT_URI, new String[]{ContactsContract.CommonDataKinds.Phone.NUMBER, ContactsContract.CommonDataKinds.Phone.TYPE}, ContactsContract.CommonDataKinds.Phone.CONTACT_ID + "=" + id, null, null, null);
//                while (inCur.moveToNext()) {
//                    int number = inCur.getInt(inCur.getColumnIndex(ContactsContract.CommonDataKinds.Phone.NUMBER));
//                    int type = inCur.getInt(inCur.getColumnIndex(ContactsContract.CommonDataKinds.Phone.TYPE));
//                    stringBuilder.append("number : " + number + "type" + type);
//                }
//                inCur.close();
//            }
//        }
//        cursor.close();
//        textView.setText(stringBuilder.toString());
        /**
         * 插入联系人
         * */
        ContentValues values = new ContentValues();
        /**
         * 先查询raw_contacts表中的主键，然后将主键加1就是插入联系人的_id
         * */
        Cursor cursorY = contentResolver.query(Uri.parse("content://com.android.contacts/raw_contacts"), new String[]{"_id"}, null, null, null);
        int contact_id = 1;
        if (cursorY.moveToNext()) {
            int _id = cursorY.getInt(0);
            contact_id = ++_id;
        }
        values.put("contact_id", contact_id);
        contentResolver.insert(Uri.parse("content://com.android.contacts/raw_contacts"), values);
        values.clear();

        values.put("data1", "sb");
        values.put("mimetype", "vnd.android.cursor.item/name");
        values.put("raw_contact_id" , contact_id);
        contentResolver.insert(Uri.parse("content://com.android.contacts/data") , values);
        values.clear();

        values.put("data1" , "138438");
        values.put("mimetype" , "vnd.android.cursor.item/phone_v2");
        values.put("raw_contact_id" , contact_id);
        contentResolver.insert(Uri.parse("content://com.android.contacts/data") , values);
    }
}
```
