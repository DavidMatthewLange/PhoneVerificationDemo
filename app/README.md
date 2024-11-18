# PhoneVerificationDemo App

A demonstration on how to verify user accounts with phone verification during account setup.

## Steps:
1. Setting up Firebase.

- Setup Firebase for the application using the Firebase extension.
- Ensure the google-services.json file is in the /app directory.
- Enable both Email/Password and Phone authentication in the Firebase console.

2. Update the libraries and dependencies

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

3. Build the Authorization Screen

- Create a Kotlin file to handle the Authorization Screen. 
- Code is implemented below:

  - @Composable
    fun AuthScreen(authViewModel: AuthViewModel = viewModel()) {
    val email = remember { mutableStateOf("") }
    val password = remember { mutableStateOf("") }
    val phoneNumber = remember { mutableStateOf("") }
    val verificationCode = remember { mutableStateOf("") }
    val isVerificationSent = authViewModel.isVerificationSent.collectAsState()

    Column(
    modifier = Modifier
    .fillMaxSize()
    .padding(16.dp),
    verticalArrangement = Arrangement.Center,
    horizontalAlignment = Alignment.CenterHorizontally
    ) {
    if (!isVerificationSent.value) {
    TextField(value = email.value, onValueChange = { email.value = it }, label = { Text("Email") })
    Spacer(modifier = Modifier.height(8.dp))
    TextField(value = password.value, onValueChange = { password.value = it }, label = { Text("Password") })
    Spacer(modifier = Modifier.height(8.dp))
    TextField(value = phoneNumber.value, onValueChange = { phoneNumber.value = it }, label = { Text("Phone Number") })
    Spacer(modifier = Modifier.height(16.dp))
    Button(onClick = {
    authViewModel.createAccount(email.value, password.value, phoneNumber.value)
    }) {
    Text("Create Account")
    }
    } else {
    TextField(value = verificationCode.value, onValueChange = { verificationCode.value = it }, label = { Text("Verification Code") })
    Spacer(modifier = Modifier.height(16.dp))
    Button(onClick = { authViewModel.verifyPhoneNumber(verificationCode.value) }) {
    Text("Verify")
    }
    }
    }
    }

4. Build the Firebase functionality.

- Create a new Kotlin Class file to handle all Firebase functionality.
- Code is implemented below:

  - class AuthViewModel : ViewModel() {
    private val _isVerificationSent = MutableStateFlow(false)
    val isVerificationSent: StateFlow<Boolean> = _isVerificationSent

    fun createAccount(email: String, password: String, phoneNumber: String) {
    viewModelScope.launch {
    FirebaseUtils.createAccount(email, password, phoneNumber) {
    _isVerificationSent.value = it
    }
    }
    }

    fun verifyPhoneNumber(code: String) {
    FirebaseUtils.verifyPhoneNumber(code) { success ->
    if (success) {
    // Handle successful phone verification
    }
    }
    }
    }

- Create a Kotlin Object to handle Firebase Utilities
- Code is implemented below:
  - object FirebaseUtils {
    private val auth = FirebaseAuth.getInstance()
    private var verificationId: String? = null

    fun createAccount(email: String, password: String, phoneNumber: String, onComplete: (Boolean) -> Unit) {
    auth.createUserWithEmailAndPassword(email, password)
    .addOnCompleteListener { task ->
    if (task.isSuccessful) {
    sendVerificationCode(phoneNumber, onComplete)
    } else {
    Log.e("FirebaseUtils", "Error creating account: ${task.exception}")
    onComplete(false)
    }
    }
    }

    private fun sendVerificationCode(phoneNumber: String, onComplete: (Boolean) -> Unit) {
    val options = PhoneAuthProvider.OnVerificationStateChangedCallbacks(
    onVerificationCompleted = { credential ->
    // Auto-retrieval happens here
    onComplete(true)
    },
    onVerificationFailed = { e ->
    Log.e("FirebaseUtils", "Verification failed: $e")
    onComplete(false)
    },
    onCodeSent = { id, _ ->
    verificationId = id
    onComplete(true)
    }
    )
    PhoneAuthProvider.verifyPhoneNumber(
    PhoneAuthOptions.newBuilder(auth)
    .setPhoneNumber(phoneNumber)
    .setTimeout(60L, TimeUnit.SECONDS)
    .setActivity(/* Pass the activity */)
    .setCallbacks(callbacks)
    .build()
    )
    }

    fun verifyPhoneNumber(code: String, onComplete: (Boolean) -> Unit) {
    val credential = PhoneAuthProvider.getCredential(verificationId ?: "", code)
    auth.signInWithCredential(credential)
    .addOnCompleteListener { task ->
    onComplete(task.isSuccessful)
    }
    }
    }

5. Call the function

- In the Main Activity, call the AuthScreen() function

6. Verify everything works
   - Launch the app, enter the email, password, and phone number.
   - A text message will be sent to the user with a verification code.
   - Enter the code to verify the phone number.