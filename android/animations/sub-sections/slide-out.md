# How to Make Slide Out Animations

**<u>`slide-out-left.xml`</u>**

```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">

    <translate android:fromXDelta="0%" android:toXDelta="-100%"
        android:fromYDelta="0%" android:toYDelta="0%"
        android:duration="400"/>
</set>
```

**<u>`slide-out-right.xml`</u>**

```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">

    <translate android:fromXDelta="0%" android:toXDelta="100%"
        android:fromYDelta="0%" android:toYDelta="0%"
        android:duration="400"/>
</set>
```

---

## Change Log

- [04-10-2020] - Added Slide Out Left and Right