# Experiment 8: Implementation of Menus and WebView in Android

**Student Name:** Gayathri R  
**USN:** 25MCAR0093  
**Course:** Mobile Application Development Laboratory  
**Target Environment:** Android Studio (Kotlin DSL), Android SDK (API 24 to 34+), Google Pixel 5 / Pixel Emulators  

---

## 1. Project Overview & Concepts

### 1.1 Objective
To demonstrate the implementation of user interface options menus and embedded web browsing using Android's native **WebView** and **WebViewClient** components.

### 1.2 Theoretical Concepts
In modern Android application design, presenting web content and secondary actions seamlessly requires combining native menu frameworks with embedded rendering engines:
- **`WebView` (Browser View):** An embedded browser view powered by the Chromium engine that allows remote web pages or local HTML content to render directly inside an activity layout without launching an external browser app.
- **`WebViewClient` (Navigation Interceptor):** Intercepts URL loading events so that link navigation is contained within the application's WebView container instead of delegating to default external browser intents (such as Google Chrome).
- **`WebSettings`:** Configures runtime properties of the browser component, notably enabling `javaScriptEnabled = true` to allow interactive web features to execute.
- **Options Menu (`res/menu`):** Provides a standardized overflow menu located in the top Action Bar / App Bar. It is defined in XML, inflated via `MenuInflater` inside `onCreateOptionsMenu()`, and actions are captured in `onOptionsItemSelected()`.
- **`OnBackPressedDispatcher`:** Intercepts system back-button presses to traverse through internal browser history (`webView.goBack()`) using `webView.canGoBack()` rather than prematurely terminating the Activity.
- **Internet Permission:** Declared via `<uses-permission android:name="android.permission.INTERNET" />` inside `AndroidManifest.xml` to grant network access.

### 1.3 Implementation Scenario
The application features a dedicated student header banner displaying **Gayathri R** and **USN: 25MCAR0093** positioned directly above an embedded `WebView`. Upon launch, Google loads by default. The top-right Options Menu provides choices to switch between **Google**, **Wikipedia**, and **GitHub**, or to trigger a **Refresh** of the current page. Each action provides user feedback via a `Toast` message and directs the `WebView` to the requested URL.

---

## 2. Project Directory & File Structure

```text
MenuWebViewDemo/
├── app/
│   ├── manifests/
│   │   └── AndroidManifest.xml          # Declares INTERNET permission and Activity registration
│   ├── java/com/example/menuwebviewdemo/
│   │   └── MainActivity.kt              # Inflates menu, binds actions, handles back navigation
│   └── res/
│       ├── layout/
│       │   └── activity_main.xml        # Layout containing the student header banner & WebView
│       ├── menu/
│       │   └── main_menu.xml            # Options Menu schema (Google, Wikipedia, GitHub, Refresh)
│       ├── values/
│       │   ├── colors.xml               # Theme palette definitions
│       │   ├── strings.xml              # Application strings
│       │   └── themes.xml               # Material design themes
│       └── mipmap/                      # Application launcher icons
├── build.gradle.kts (Project)
└── app/build.gradle.kts (Module)        # Module configuration & dependencies
```

---

## 3. Core Source Code

### 3.1 Manifest Declaration (`app/src/main/AndroidManifest.xml`)
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <!-- Mandatory permission for loading remote web pages -->
    <uses-permission android:name="android.permission.INTERNET" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="MenuWebViewDemo"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.MaterialComponents.DayNight.DarkActionBar"
        android:usesCleartextTraffic="true">

        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

### 3.2 Main Layout with Student Info (`app/src/main/res/layout/activity_main.xml`)
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/mainLayout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <!-- Student Header Banner -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:background="#E8F0FE"
        android:padding="12dp">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Student Name: Gayathri R"
            android:textSize="15sp"
            android:textStyle="bold"
            android:textColor="#1A73E8" />

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="2dp"
            android:text="USN: 25MCAR0093"
            android:textSize="13sp"
            android:textColor="#3C4043" />
    </LinearLayout>

    <!-- Embedded Browser Component -->
    <WebView
        android:id="@+id/webView"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1" />

</LinearLayout>
```

### 3.3 Menu Resource File (`app/src/main/res/menu/main_menu.xml`)
```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <item
        android:id="@+id/menu_google"
        android:title="Google"
        app:showAsAction="never" />

    <item
        android:id="@+id/menu_wikipedia"
        android:title="Wikipedia"
        app:showAsAction="never" />

    <item
        android:id="@+id/menu_github"
        android:title="GitHub"
        app:showAsAction="never" />

    <item
        android:id="@+id/menu_refresh"
        android:title="Refresh"
        app:showAsAction="never" />

</menu>
```

### 3.4 Main Logic (`app/src/main/java/com/example/menuwebviewdemo/MainActivity.kt`)
```kotlin
package com.example.menuwebviewdemo

import android.os.Bundle
import android.view.Menu
import android.view.MenuItem
import android.webkit.WebView
import android.webkit.WebViewClient
import android.widget.Toast
import androidx.activity.OnBackPressedCallback
import androidx.appcompat.app.AppCompatActivity

class MainActivity : AppCompatActivity() {

    private lateinit var myWebView: WebView

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        myWebView = findViewById(R.id.webView)

        // Enable JavaScript execution inside WebView
        myWebView.settings.javaScriptEnabled = true

        // Force all links to open inside this WebView
        myWebView.webViewClient = WebViewClient()

        // Load initial default web page
        myWebView.loadUrl("https://www.google.com")

        // Intercept back key to navigate through WebView web history
        onBackPressedDispatcher.addCallback(this, object : OnBackPressedCallback(true) {
            override fun handleOnBackPressed() {
                if (myWebView.canGoBack()) {
                    myWebView.goBack()
                } else {
                    isEnabled = false
                    onBackPressedDispatcher.onBackPressed()
                }
            }
        })
    }

    // Inflate Options Menu from XML
    override fun onCreateOptionsMenu(menu: Menu?): Boolean {
        menuInflater.inflate(R.menu.main_menu, menu)
        return true
    }

    // Handle Option Item Clicks
    override fun onOptionsItemSelected(item: MenuItem): Boolean {
        return when (item.itemId) {
            R.id.menu_google -> {
                myWebView.loadUrl("https://www.google.com")
                Toast.makeText(this, "Loading Google...", Toast.LENGTH_SHORT).show()
                true
            }
            R.id.menu_wikipedia -> {
                myWebView.loadUrl("https://www.wikipedia.org")
                Toast.makeText(this, "Loading Wikipedia...", Toast.LENGTH_SHORT).show()
                true
            }
            R.id.menu_github -> {
                myWebView.loadUrl("https://www.github.com")
                Toast.makeText(this, "Loading GitHub...", Toast.LENGTH_SHORT).show()
                true
            }
            R.id.menu_refresh -> {
                myWebView.reload()
                Toast.makeText(this, "Refreshing...", Toast.LENGTH_SHORT).show()
                true
            }
            else -> super.onOptionsItemSelected(item)
        }
    }
}
```

---

## 4. Test Cases and Execution Results

### 4.1 Test Case Matrix

| Test ID | Test Scenario | Trigger Action | Expected Outcome | Status |
| :--- | :--- | :--- | :--- | :--- |
| **TC_01** | Student Identity & Default Page Load | Launch application | Top banner displays **Gayathri R** and **25MCAR0093**; WebView renders Google homepage | PASS |
| **TC_02** | Options Menu Navigation (Wikipedia) | Tap 3 dots $\rightarrow$ Select "Wikipedia" | Toast displays `"Loading Wikipedia..."`; WebView loads `https://www.wikipedia.org` | PASS |
| **TC_03** | Menu Navigation & History Back-Press | Select "GitHub", click a link, then press device Back button | Toast displays `"Loading GitHub..."`; back key navigates to previous webpage within app | PASS |




## 5. How to Run This Project
1. Clone the repository:
   ```bash
   git clone https://github.com/<your-username>/MenuWebViewDemo.git
   ```
2. Open the project folder in **Android Studio**.
3. Allow Gradle to build and sync dependencies.
4. Launch an emulator (e.g., Pixel 5 with API 24+) or connect a physical Android device with USB debugging enabled.
5. Click **Run** (`Shift + F10`) to compile and launch the application.
