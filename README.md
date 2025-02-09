### **Issue Title**
Auto-generated `capacitor.build.gradle` forces Java 21, causing build failures in environments with Java 17

---

### **Description**
When running the command:
```bash
ionic cap run android
```
the build process fails with the following error:
```
> Task :capacitor-android:compileDebugJavaWithJavac FAILED
FAILURE: Build failed with an exception.
* What went wrong:
Execution failed for task ':capacitor-android:compileDebugJavaWithJavac'.
> error: invalid source release: 21
```

This error occurs because the auto-generated `capacitor.build.gradle` file explicitly sets the `sourceCompatibility` and `targetCompatibility` to `JavaVersion.VERSION_21`. However, my development environment is configured to use Java 17, which is incompatible with this setting.

---

### **Steps to Reproduce**
1. Set up a project using Capacitor (version 7.x).
2. Configure the environment to use Java 17:
   ```bash
   java -version
   ```
   Output:
   ```
   openjdk version "17.0.14" 2025-01-21
   OpenJDK Runtime Environment (build 17.0.14+7-Ubuntu-124.04)
   OpenJDK 64-Bit Server VM (build 17.0.14+7-Ubuntu-124.04, mixed mode, sharing)
   ```
3. Run the following commands:
   ```bash
   ionic cap sync android
   ionic cap run android
   ```
4. Observe the build failure with the error:
   ```
   error: invalid source release: 21
   ```

---

### **Expected Behavior**
The `capacitor.build.gradle` file should respect the Java version available in the development environment or allow developers to configure the desired Java version via an environment variable or configuration option.

---

### **Actual Behavior**
The `capacitor.build.gradle` file is auto-generated with the following configuration:
```groovy
android {
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_21
        targetCompatibility JavaVersion.VERSION_21
    }
}
```
This forces the build process to use Java 21, regardless of the environment's Java version.

---

### **Environment Details**
- **Operating System:** Linux (Ubuntu 6.8.0-52-generic amd64)
- **Java Version:**
  ```bash
  java -version
  ```
  Output:
  ```
  openjdk version "17.0.14" 2025-01-21
  OpenJDK Runtime Environment (build 17.0.14+7-Ubuntu-124.04)
  OpenJDK 64-Bit Server VM (build 17.0.14+7-Ubuntu-124.04, mixed mode, sharing)
  ```
- **Gradle Version:**
  ```bash
  gradle -v
  ```
  Output:
  ```
  Gradle 8.5 (later updated to 8.9 in some configurations)
  ```
- **Capacitor Version:** 7.x (as specified in `package.json`)
- **Node.js Version:** v18.x (or your specific version)

---

### **Attempts to Resolve**
1. **Set `OVERRIDE_JAVA_VERSION`:**
   - Exported the environment variable:
     ```bash
     export OVERRIDE_JAVA_VERSION=17
     ```
   - Result: The auto-generated `capacitor.build.gradle` still uses Java 21.

2. **Edit `gradle.properties`:**
   - Added the following line to `android/gradle.properties`:
     ```properties
     org.gradle.java.home=/usr/lib/jvm/java-17-openjdk-amd64
     ```
   - Result: While Gradle uses Java 17, the `capacitor.build.gradle` file still enforces Java 21.

3. **Modify `variables.gradle`:**
   - Updated `android/variables.gradle` to include:
     ```groovy
     ext {
         javaVersion = JavaVersion.VERSION_17
     }
     ```
   - Result: The `capacitor.build.gradle` file ignores this setting.

4. **Manually Edit `capacitor.build.gradle`:**
   - Changed the Java version to 17:
     ```groovy
     compileOptions {
         sourceCompatibility JavaVersion.VERSION_17
         targetCompatibility JavaVersion.VERSION_17
     }
     ```
   - Result: Changes are overwritten during subsequent `ionic cap sync` operations.

---

### **Proposed Solutions**
1. **Add a Configuration Option:**
   - Allow developers to specify the desired Java version via an environment variable (e.g., `OVERRIDE_JAVA_VERSION`) or a configuration file.

2. **Auto-Detect Java Version:**
   - Automatically detect the Java version available in the environment and adjust the `sourceCompatibility` and `targetCompatibility` settings accordingly.

3. **Documentation Update:**
   - Clearly document how to override the Java version in environments where Java 21 is not available.

---

### **Temporary Workaround**
As a temporary solution, I am using a post-sync script to patch the `capacitor.build.gradle` file after every `ionic cap sync` operation. Example script:
```javascript
const fs = require('fs');
const path = require('path');

const filePath = path.join(__dirname, 'android', 'capacitor.build.gradle');

try {
  let content = fs.readFileSync(filePath, 'utf8');
  content = content.replace(/JavaVersion\.VERSION_21/g, 'JavaVersion.VERSION_17');
  fs.writeFileSync(filePath, content, 'utf8');
  console.log('Successfully patched capacitor.build.gradle to use Java 17.');
} catch (error) {
  console.error('Failed to patch capacitor.build.gradle:', error.message);
}
```

---

### **Additional Notes**
- This issue affects developers who cannot upgrade to Java 21 due to environmental constraints or compatibility requirements.
- A similar issue was reported in [GitHub Issue #XXXX](link-to-related-issue), but it remains unresolved.

---

### **Request**
Please consider implementing a feature or fix to address this issue, as it impacts the usability of Capacitor in environments with Java versions other than 21.

Thank you for your attention to this matter!

--- 
