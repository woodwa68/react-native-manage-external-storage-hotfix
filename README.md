# react-native-manage-external-storage-hotfix

react native package to promt user to allow access to manage all files
## Introduction
This Package (react-native-manage-external-storage-hotfix) is implemented in java and it is register in react native and a native module. The package solves the issues about implementing **MANAGE_EXTERNAL_STORAGE** Permission that is need to access all files in an android phone in React Native 

The package is developed as a result of changes in Google Play Store Privacy Policy and the need to implement  MANAGE_EXTERNAL_STORAGE Permission in React Native. Google introduced this Permission from Android 11 (API level 30) or higher and its the ONLY way to access and Manage All files.

This is how this package implements the permission. It implements by promting the user to allow the app to access and manage all files.
![Permission Access](./src/img/access.jpg?raw=true "Title")

## Installation

```sh
npm install react-native-manage-external-storage-hotfix
```
## Android Setup
### 1. First Go to ---
*<---Project directory---/>/android/app/src/main/AndroidManifest.xml*
then add the following perssions
```
    <uses-permission android:name="android.permission.MANAGE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.DOWNLOAD_WITHOUT_NOTIFICATION" />
```

### 2.1 Second Go to ----
*<----Your Projectdirectory----/>/android/app/src/main/java/com/<---ProjectName---/>*
then create a file called  **PermissionFileModule.java** and paste the following code

```
package managePermission;

import static android.Manifest.permission.WRITE_EXTERNAL_STORAGE;
import static android.Manifest.permission.READ_EXTERNAL_STORAGE;

import android.os.Build;
import android.os.Environment;
import android.content.pm.PackageManager;
import android.content.Intent;
import android.widget.Toast;
import android.app.Activity;
import android.net.Uri;
import android.provider.Settings;

import androidx.annotation.Nullable;
import androidx.annotation.NonNull;
import androidx.core.content.ContextCompat;
import androidx.core.app.ActivityCompat;
import androidx.core.content.ContextCompat;

import com.facebook.react.bridge.Callback;
import com.facebook.react.bridge.ActivityEventListener;
import com.facebook.react.bridge.ReactContextBaseJavaModule;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.bridge.ReactMethod;
import com.facebook.react.uimanager.IllegalViewOperationException;

public class PermissionFileModule extends ReactContextBaseJavaModule implements ActivityEventListener {

    public PermissionFileModule(@Nullable ReactApplicationContext reactContext) {
        super(reactContext);
        reactContext.addActivityEventListener(this);
    }

    @NonNull
    @Override
    public String getName() {
        return "PermissionFile";
    }

    @ReactMethod
    public void checkAndGrantPermission(Callback errorCallback, Callback successCallback) {
        try {
            if (!checkPermission()) {
                requestPermission();
                successCallback.invoke(false);
            } else {
                successCallback.invoke(true);
            }
        } catch (IllegalViewOperationException e) {
            errorCallback.invoke(e.getMessage());
        }
    }

    private boolean checkPermission() {
        if (Build.VERSION.SDK_INT >= 30) {
            return Environment.isExternalStorageManager();
        } else {
            int result = ContextCompat.checkSelfPermission(getReactApplicationContext(), READ_EXTERNAL_STORAGE);
            int result1 = ContextCompat.checkSelfPermission(getReactApplicationContext(), WRITE_EXTERNAL_STORAGE);
            return result == PackageManager.PERMISSION_GRANTED && result1 == PackageManager.PERMISSION_GRANTED;
        }
    }

    private void requestPermission() {
        if (Build.VERSION.SDK_INT >= 30) {
            try {
                Intent intent = new Intent(Settings.ACTION_MANAGE_APP_ALL_FILES_ACCESS_PERMISSION);
                intent.addCategory("android.intent.category.DEFAULT");
                intent.setData(Uri.parse(String.format("package:%s",getReactApplicationContext().getPackageName())));
                getCurrentActivity().startActivityForResult(intent, 2296);
            } catch (Exception e) {
                Intent intent = new Intent();
                intent.setAction(Settings.ACTION_MANAGE_ALL_FILES_ACCESS_PERMISSION);
                getCurrentActivity().startActivityForResult(intent, 2296);
            }
        } else {
            //below android 11
            ActivityCompat.requestPermissions(getCurrentActivity(), new String[]{WRITE_EXTERNAL_STORAGE}, 100);
        }
    }

    @Override
    public void onActivityResult(Activity activity, int requestCode, int resultCode, Intent data) {
        if (requestCode == 2296) {
            if (Build.VERSION.SDK_INT >= 30) {
                if (Environment.isExternalStorageManager()) {
                    Toast.makeText(getReactApplicationContext(), "Access granted", Toast.LENGTH_SHORT).show();
                } else {
                    Toast.makeText(getReactApplicationContext(), "Access not granted", Toast.LENGTH_SHORT).show();
                }
            }
        }
    }

    @Override
    public void onNewIntent(Intent intent) {
        // do nothing
    }
}
```
### 2.3 Third Go to ----
*<----Your Projectdirectory----/>/android/app/src/main/java/com/<---ProjectName---/>*
then create a file called  **PermissionFilePackage.java** and paste the following code
```
package managePermission;

import androidx.annotation.NonNull;

import com.facebook.react.ReactPackage;
import com.facebook.react.bridge.NativeModule;
import com.facebook.react.uimanager.ViewManager;
import com.facebook.react.bridge.ReactApplicationContext;

import java.util.ArrayList;
import java.util.List;
import java.util.Collections;

import managePermission.PermissionFileModule;

public class PermissionFilePackage implements ReactPackage {
    @NonNull
    @Override
    public List<NativeModule> createNativeModules(@NonNull ReactApplicationContext reactContext) {
        List<NativeModule> modules = new ArrayList<>();

        modules.add(new PermissionFileModule(reactContext));
        return modules;
    }

    @NonNull
    @Override
    public List<ViewManager> createViewManagers(@NonNull ReactApplicationContext reactContext) {
        return Collections.emptyList();
    }
}
```

### 2.4 Lastly Go to ----
*<----Your Projectdirectory----/>/android/app/src/main/java/com/<---ProjectName---/>/MainApplication*
**at the top of the file add**
```
 import managePermission.PermissionFilePackage; <!-- import this -->
```

**then update this part**
```
   <!-- Other codes -->
    @Override
        protected List<ReactPackage> getPackages() {
          @SuppressWarnings("UnnecessaryLocalVariable")
          List<ReactPackage> packages = new PackageList(this).getPackages();
          // Packages that cannot be autolinked yet can be added manually here, for example:
          // packages.add(new MyReactNativePackage());
          packages.add(new PermissionFilePackage());  <!-- add this line -->
          return packages;
        }

        <!-- Other codes -->

```

## Usage

```js
import  ManageExternalStorage  from 'react-native-manage-external-storage-hotfix';

// ...Other codes

const [result, setResult] = useState(false);
 useEffect(() => {
    async function AskPermission() {
    await ManageExternalStorage.checkAndGrantPermission(
           err => { 
             setResult(false)
          },
          res => {
           setResult(true)
          },
        )
   }
     AskPermission()  // This function is only executed once if the user allows the permission and this package retains that permission 
  }, []);

// .... Other codes

```

## Contributing

See the [contributing guide](CONTRIBUTING.md) to learn how to contribute to the repository and the development workflow.

## License

MIT

---

## Author
 ### Name  :  **Abdiladif Hassan**
 
 ### Email  :   **abdiladifhassan114@gmail.com  || abdiladifhassan115@gmail.com**
 ### Website :  **https://hassanjr.vercel.app**
