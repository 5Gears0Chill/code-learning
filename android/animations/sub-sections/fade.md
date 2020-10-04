# How to Make Fade Animations

**<u>`fade-in.xml`</u>**

```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <alpha
        android:duration="@android:integer/config_mediumAnimTime"
        android:fromAlpha="0.0"
        android:toAlpha="1.0"/>
</set>
```

**<u>`fade-out.xml`</u>**

```xml
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <alpha
        android:duration="@android:integer/config_mediumAnimTime"
        android:fromAlpha="1.0"
        android:toAlpha="0.0"/>
</set>
```

---

## Change Log

- [04-10-2020] - Added Fade In and Out