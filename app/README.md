# PhoneVerificationDemo App

A demonstration on how to verify user accounts with phone verification during account setup.

## Steps:
- Setup Firebase for the application using the Firebase extension.
- Ensure the google-services.json file is in the /app directory.
- Enable both Email/Password and Phone authentication in the Firebase console.

- Add versions to the libs.versions.toml file (if not implemented by Firebase)
  - firebase-bom = "32.3.1"
    lifecycle-runtime = "2.6.1"
    compose = "1.6.0"
  
- Add libraries to the libs.versions.toml file (if not implemented by Firebase)
  - firebase-bom = { module = "com.google.firebase:firebase-bom", version.ref = "firebase-bom" }
    firebase-auth = { module = "com.google.firebase:firebase-auth-ktx" }
    firebase-database = { module = "com.google.firebase:firebase-database-ktx" }
    lifecycle-runtime = { module = "androidx.lifecycle:lifecycle-runtime-ktx", version.ref = "lifecycle-runtime" }
    compose-ui = { module = "androidx.compose.ui:ui", version.ref = "compose" }
    compose-material = { module = "androidx.compose.material:material", version.ref = "compose" }
  
- Add dependencies to the build.gradle.kts file (if not implemented by Firebase)
  - implementation (platform(libs.firebase.bom))    // Firebase BOM for managing Firebase versions
    implementation (libs.firebase.auth)             // Firebase Authentication
    implementation (libs.firebase.database)         // Firebase Realtime Database
    implementation (libs.lifecycle.runtime)         // Lifecycle runtime for Compose
    implementation (libs.compose.ui)                // Compose UI
    implementation (libs.compose.material)          // Compose Material

- Sync the project to ensure all libraries and dependencies load correctly.

- In the Main Activity, call the AuthScreen()