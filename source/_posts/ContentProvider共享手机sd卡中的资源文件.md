---
title: ContentProvider共享手机sd卡中的资源文件
date: 2019-05-17 19:55:08
tags: android
---
ContentProvider通过uri来标识其它应用要访问的数据，通过ContentResolver的增、删、改、查方法实现对共享数据的操作。还可以通过注册ContentObserver来监听数据是否发生了变化来对应的刷新页面。

通过ContentResolver加载手机中的文件
``` bash
abstract class LocalStorageLoader<T> {

    protected abstract val context: Context

    /**
     * using the content:// scheme, for the content to retrieve.
     */
    protected abstract fun getUri(): Uri

    /**
     * A list of which columns to return. Passing null will
     * return all columns, which is inefficient.
     */
    protected abstract fun getProjection(): Array<String>

    /**
     * A filter declaring which rows to return, formatted as an
     * SQL WHERE clause (excluding the WHERE itself). Passing null will
     * return all rows for the given URI.
     */
    protected abstract fun getSelection(): String?

    /**
     * You may include ?s in selection, which will be
     * replaced by the values from selectionArgs, in the order that they
     * appear in the selection. The values will be bound as Strings.
     */
    protected abstract fun getSelectionArgs(): Array<String>?

    /**
     * How to order the rows, formatted as an SQL ORDER BY
     * clause (excluding the ORDER BY itself). Passing null will use the
     * default sort order, which may be unordered.
     */
    protected abstract fun getSortOrder(): String?

    protected abstract fun parse(cursor: Cursor): T?

    fun query(): List<T> {
        val models = ArrayList<T>()
        val resolver = context.contentResolver
        val cursor = resolver.query(getUri(), getProjection(), getSelection(), getSelectionArgs(), getSortOrder())
        cursor?.let {
            while (it.moveToNext()) {
                val data = parse(cursor)
                if (null != data) {
                    models.add(data)
                }
            }
            if (!it.isClosed) {
                it.close()
            }
        }
        return models
    }
}

```
加载sd卡中的所有jpg,png,gif图片
``` bash
class DefaultImageLoader(override val context: Context) : LocalStorageLoader<FileInfo>() {

        override fun getUri(): Uri {
            return MediaStore.Images.Media.EXTERNAL_CONTENT_URI
        }

        override fun getProjection(): Array<String> {
            return arrayOf(
                    MediaStore.Images.Media.DATA,
                    MediaStore.Images.Media.MIME_TYPE,
                    MediaStore.Images.Media.BUCKET_ID,
                    MediaStore.Images.Media.BUCKET_DISPLAY_NAME,
                    MediaStore.Images.Media.DATE_TAKEN,
                    MediaStore.Images.Media.SIZE,
                    MediaStore.Images.Media.DISPLAY_NAME
            )
        }

        override fun getSelection(): String? {
            return MediaStore.Images.Media.MIME_TYPE + "=? or " + MediaStore.Images.Media.MIME_TYPE + "=? or " + MediaStore.Images.Media.MIME_TYPE + "=?"
        }

        override fun getSelectionArgs(): Array<String>? {
            return arrayOf("image/jpeg", "image/png", "image/gif")
        }

        override fun getSortOrder(): String? {
            return MediaStore.Images.Media.DATE_TAKEN + " desc"
        }

        override fun parse(cursor: Cursor): FileInfo? {
            val localPath = cursor.getString(cursor.getColumnIndex(MediaStore.Images.Media.DATA))
            val mimeType = cursor.getString(cursor.getColumnIndex(MediaStore.Images.Media.MIME_TYPE))
            val parentId = cursor.getString(cursor.getColumnIndex(MediaStore.Images.Media.BUCKET_ID))
            val parentName = cursor.getString(cursor.getColumnIndex(MediaStore.Images.Media.BUCKET_DISPLAY_NAME))
            val dateToken = cursor.getLong(cursor.getColumnIndex(MediaStore.Images.Media.DATE_TAKEN))
            val length = cursor.getLong(cursor.getColumnIndex(MediaStore.Images.Media.SIZE))
            val displayName = cursor.getString(cursor.getColumnIndex(MediaStore.Images.Media.DISPLAY_NAME))
            if (!TextUtils.isEmpty(localPath) && length > 10240) {
                val options = BitmapFactory.Options()
                options.inJustDecodeBounds = true
                BitmapFactory.decodeFile(localPath, options)
                return FileInfo(displayName, localPath, length, 0, dateToken, options.outWidth, options.outHeight, mimeType, parentId, parentName)
            }
            return null
        }
    }
```
加载sd卡中的所有mp4视频
``` bash
class DefaultVideoLoader(override val context: Context) : LocalStorageLoader<FileInfo>() {

        override fun getUri(): Uri {
            return MediaStore.Video.Media.EXTERNAL_CONTENT_URI
        }

        override fun getProjection(): Array<String> {
            return arrayOf(
                    MediaStore.Video.Media.DATA,
                    MediaStore.Video.Media.MIME_TYPE,
                    MediaStore.Video.Media.BUCKET_ID,
                    MediaStore.Video.Media.BUCKET_DISPLAY_NAME,
                    MediaStore.Video.Media.DURATION,
                    MediaStore.Video.Media.DATE_TAKEN,
                    MediaStore.Video.Media.SIZE,
                    MediaStore.Video.Media.DISPLAY_NAME
            )
        }

        override fun getSelection(): String? {
            return MediaStore.Images.Media.MIME_TYPE + "=?"
        }

        override fun getSelectionArgs(): Array<String>? {
            return arrayOf("video/mp4")
        }

        override fun getSortOrder(): String? {
            return MediaStore.Video.Media.DATE_TAKEN + " desc"
        }

        override fun parse(cursor: Cursor): FileInfo? {
            val localPath = cursor.getString(cursor.getColumnIndex(MediaStore.Video.Media.DATA))
            val mimeType = cursor.getString(cursor.getColumnIndex(MediaStore.Video.Media.MIME_TYPE))
            val parentId = cursor.getString(cursor.getColumnIndex(MediaStore.Video.Media.BUCKET_ID))
            val parentName = cursor.getString(cursor.getColumnIndex(MediaStore.Video.Media.BUCKET_DISPLAY_NAME))
            val dateToken = cursor.getLong(cursor.getColumnIndex(MediaStore.Video.Media.DATE_TAKEN))
            val duration = cursor.getLong(cursor.getColumnIndex(MediaStore.Video.Media.DURATION))
            val length = cursor.getLong(cursor.getColumnIndex(MediaStore.Video.Media.SIZE))
            val displayName = cursor.getString(cursor.getColumnIndex(MediaStore.Video.Media.DISPLAY_NAME))
            if (!TextUtils.isEmpty(localPath) && length > 10240) {
                return FileInfo(displayName, localPath, length, duration, dateToken, 0, 0, mimeType, parentId, parentName)
            }
            return null
        }
    }
```