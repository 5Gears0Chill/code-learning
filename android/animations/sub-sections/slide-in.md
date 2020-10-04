# How to Make Slide In Animations

**<u>`slide-in-left.xml`</u>**

```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">

    <translate android:fromXDelta="-100%" android:toXDelta="0%"
        android:fromYDelta="0%" android:toYDelta="0%"
        android:duration="400"/>
</set>
```

**<u>`slide-in-right.xml`</u>**

```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">

    <translate android:fromXDelta="100%" android:toXDelta="0%"
        android:fromYDelta="0%" android:toYDelta="0%"
        android:duration="400"/>
</set>
```

---

## Change Log

- [04-10-2020] - Added Slide In Left and Right