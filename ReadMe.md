# Final Exam Questions & Answers - Appointment System

This document contains comprehensive answers to questions about OOP concepts, Polymorphism, Database, API usage, and Enum constants used in this Android Appointment System application.

---

## 📚 TABLE OF CONTENTS

1. [Object-Oriented Programming (OOP)](#object-oriented-programming-oop)
2. [Polymorphism](#polymorphism)
3. [Database Usage](#database-usage)
4. [API Usage](#api-usage)
5. [Enum Constants](#enum-constants)
6. [Additional OOP Concepts](#additional-oop-concepts)

---

## 🎯 OBJECT-ORIENTED PROGRAMMING (OOP)

### Q1: What OOP concepts did you use in this app?

**Answer:**

I implemented several OOP concepts in this application:

#### 1. **Abstraction**
- **Abstract Class**: `User` is an abstract base class located in `model/User.kt`
- **Abstract Method**: `getRoleWelcomeMessage()` is an abstract method that must be implemented by subclasses
- **Purpose**: Defines a contract that all user types must follow while hiding implementation details

**Location**: `app/src/main/java/com/example/appointmentsystem/model/User.kt`

```kotlin
abstract class User(
    var id: Int = 0,
    var name: String = "",
    var email: String = "",
    var password: String = "",
    var role: UserRole
) {
    abstract fun getRoleWelcomeMessage(): String
}
```

#### 2. **Inheritance**
- **Base Class**: `User` (abstract class)
- **Subclasses**: 
  - `Doctor` extends `User` - located in `model/Doctor.kt`
  - `Patient` extends `User` - located in `model/Patient.kt`
- **IS-A Relationship**: Both `Doctor` and `Patient` are types of `User`

**Doctor Class** (`model/Doctor.kt`):
```kotlin
class Doctor(
    id: Int = 0,
    name: String = "",
    email: String = "",
    password: String = ""
) : User(id, name, email, password, UserRole.DOCTOR) {
    var specialization: String = ""
    var clinicName: String = ""
    var clinicAddress: String? = null
    
    override fun getRoleWelcomeMessage(): String {
        return "Welcome, Dr. $name"
    }
}
```

**Patient Class** (`model/Patient.kt`):
```kotlin
class Patient(
    id: Int = 0,
    name: String = "",
    email: String = "",
    password: String = ""
) : User(id, name, email, password, UserRole.PATIENT) {
    override fun getRoleWelcomeMessage(): String {
        return "Welcome, Patient $name"
    }
}
```

#### 3. **Encapsulation**
- **Private Properties**: Database constants are private in `DatabaseHelper`
- **Public Methods**: Controlled access through public methods like `getUserById()`, `addUser()`
- **Data Hiding**: Internal database structure is hidden, only exposed through methods

**Example** (`DatabaseHelper.kt`):
```kotlin
companion object {
    private const val DATABASE_NAME = "AppointmentSystem.db"
    private const val TABLE_USERS = "users"
    // Private constants - encapsulated
}

fun getUserById(userId: Int): User? {
    // Public method - controlled access
}
```

#### 4. **Polymorphism** (See detailed section below)

#### 5. **Data Classes**
- **Appointment**: Data class for appointment entities (`model/Appointment.kt`)
- **Clinic**: Data class for clinic information (`NearestClinicsActivity.kt`)
- **Notification**: Data class for notifications (`model/Notification.kt`)

**Example**:
```kotlin
data class Appointment(
    var id: Int = 0,
    var userId: Int = 0,
    var doctorId: Int = 0,
    var date: String = "",
    var time: String = "",
    var status: AppointmentStatus = AppointmentStatus.PENDING
)
```

---

### Q2: Where did you use OOP in this app?

**Answer:**

OOP is used throughout the application:

1. **Model Layer** (`model/` package):
   - `User.kt` - Abstract base class
   - `Doctor.kt` - Subclass of User
   - `Patient.kt` - Subclass of User
   - `Appointment.kt` - Data class
   - `Notification.kt` - Data class

2. **Database Layer** (`DatabaseHelper.kt`):
   - Extends `SQLiteOpenHelper` (inheritance from Android framework)
   - Encapsulates database operations
   - Uses polymorphism when returning `User` objects (could be Doctor or Patient)

3. **Repository Pattern** (`repository/UserRepository.kt`):
   - Singleton object pattern
   - Encapsulates user state management

4. **Activity Classes**:
   - `BaseActivity.kt` - Base class for activities (inheritance)
   - `DashboardActivity.kt` - Uses polymorphism with User objects
   - All activities extend `AppCompatActivity` (inheritance from Android framework)

5. **Adapter Classes**:
   - `DoctorsAdapter.kt` - Extends `RecyclerView.Adapter`
   - `AppointmentsAdapter.kt` - Extends `RecyclerView.Adapter`
   - `NotificationAdapter.kt` - Extends `RecyclerView.Adapter`
   - `ClinicsAdapter.kt` - Extends `RecyclerView.Adapter`
   - All demonstrate inheritance and encapsulation

---

## 🔄 POLYMORPHISM

### Q3: What is Polymorphism?

**Answer:**

**Polymorphism** means "many forms" - it allows objects of different types to be treated as objects of a common base type. The same method call can behave differently depending on the actual object type at runtime.

**Types of Polymorphism:**

1. **Runtime Polymorphism (Dynamic Binding)**: Method resolution happens at runtime based on the actual object type
2. **Compile-time Polymorphism (Method Overloading)**: Multiple methods with same name but different parameters

---

### Q4: What kind of Polymorphism did you use in this app?

**Answer:**

I used **Runtime Polymorphism (Method Overriding)** in this application.

**How it works:**
- The base class `User` defines an abstract method `getRoleWelcomeMessage()`
- Subclasses `Doctor` and `Patient` override this method with their own implementations
- At runtime, when we call `getRoleWelcomeMessage()` on a `User` reference, the correct implementation is called based on whether the object is actually a `Doctor` or `Patient`

---

### Q5: Where did you use Polymorphism in this app?

**Answer:**

Polymorphism is primarily used in the following locations:

#### 1. **DashboardActivity.kt** (Main Polymorphism Demonstration)

**Location**: `app/src/main/java/com/example/appointmentsystem/DashboardActivity.kt`

**Code Example** (Lines 73-117):
```kotlin
// Load user from database - could be Doctor or Patient
val currentUser: User? = dbHelper.getUserById(userId)

if (currentUser != null) {
    // POLYMORPHISM DEMONSTRATION
    // currentUser is of type User, but it could be either Doctor or Patient
    // When we call getRoleWelcomeMessage(), the correct implementation
    // is called based on the actual type (Doctor or Patient)
    // This is runtime polymorphism - the method is resolved at runtime
    tvWelcome.text = currentUser.getRoleWelcomeMessage()
    
    // Determine user role
    isDoctor = currentUser is Doctor
    
    // Show appropriate dashboard based on role
    if (isDoctor) {
        setupDoctorDashboard(currentUser as Doctor)
    } else {
        setupPatientDashboard()
    }
}
```

**What happens:**
- `getUserById()` returns a `User` type, but it could be a `Doctor` or `Patient`
- When `getRoleWelcomeMessage()` is called:
  - If it's a `Doctor` object → returns "Welcome, Dr. [Name]"
  - If it's a `Patient` object → returns "Welcome, Patient [Name]"
- The correct method is called automatically at runtime

#### 2. **DatabaseHelper.kt** (Polymorphism in Database Operations)

**Location**: `app/src/main/java/com/example/appointmentsystem/DatabaseHelper.kt`

**Code Example** (Lines 384-425):
```kotlin
fun getUserById(userId: Int): User? {
    // ... database query code ...
    
    return if (role == "doctor") {
        val doctor = Doctor(id, name, email, password)
        doctor.specialization = specialization ?: ""
        doctor.clinicName = clinicName ?: ""
        doctor.clinicAddress = clinicAddress
        doctor  // Returns Doctor object as User type
    } else {
        Patient(id, name, email, password)  // Returns Patient object as User type
    }
}
```

**What happens:**
- Method returns `User?` type
- Can return either `Doctor` or `Patient` object
- Both are treated as `User` type (polymorphism)

**Code Example** (Lines 226-272):
```kotlin
fun checkUser(email: String, password: String): User? {
    // ... database query code ...
    
    return if (role == "doctor") {
        val doctor = Doctor(id, name, userEmail, userPassword)
        doctor.specialization = specialization ?: ""
        doctor.clinicName = clinicName ?: ""
        doctor.clinicAddress = clinicAddress
        doctor  // Returns Doctor as User
    } else {
        Patient(id, name, userEmail, userPassword)  // Returns Patient as User
    }
}
```

#### 3. **AppointmentsAdapter.kt** (Polymorphism with Type Checking)

**Location**: `app/src/main/java/com/example/appointmentsystem/AppointmentsAdapter.kt`

**Code Example** (Lines 89-132):
```kotlin
// Patient view: Show doctor name, specialization, clinic and avatar
val doctor: User? = try {
    if (doctorId > 0) dbHelper.getUserById(doctorId) else null
} catch (e: Exception) {
    null
}

if (doctor is Doctor) {  // Type checking (polymorphism)
    val doctorName = doctor.name
    val specialization = doctor.specialization  // Doctor-specific property
    val clinicName = doctor.clinicName  // Doctor-specific property
    
    // Use doctor-specific properties
    holder.tvDoctorName.text = "Doctor: $doctorName"
    // ... more code ...
}
```

**What happens:**
- `getUserById()` returns `User?` type
- We use `is Doctor` to check if it's actually a `Doctor` instance
- If true, we can access `Doctor`-specific properties like `specialization` and `clinicName`

---

### Q6: Explain the Polymorphism flow in your app

**Answer:**

**Complete Polymorphism Flow:**

1. **User Registration/Login** (`LoginActivity.kt`):
   ```kotlin
   val user = dbHelper.checkUser(email, password)  // Returns User? (could be Doctor or Patient)
   ```

2. **Database Returns Polymorphic Object** (`DatabaseHelper.kt`):
   ```kotlin
   fun checkUser(...): User? {
       return if (role == "doctor") {
           Doctor(...)  // Returns Doctor as User type
       } else {
           Patient(...)  // Returns Patient as User type
       }
   }
   ```

3. **Polymorphic Method Call** (`DashboardActivity.kt`):
   ```kotlin
   val currentUser: User = dbHelper.getUserById(userId)
   tvWelcome.text = currentUser.getRoleWelcomeMessage()  
   // Runtime decides: Doctor.getRoleWelcomeMessage() or Patient.getRoleWelcomeMessage()
   ```

4. **Method Resolution at Runtime**:
   - If `currentUser` is actually a `Doctor` → calls `Doctor.getRoleWelcomeMessage()` → "Welcome, Dr. [Name]"
   - If `currentUser` is actually a `Patient` → calls `Patient.getRoleWelcomeMessage()` → "Welcome, Patient [Name]"

**Benefits:**
- Single interface (`User`) for multiple types (`Doctor`, `Patient`)
- Code reusability - same code works for both types
- Easy to extend - can add new user types without changing existing code
- Cleaner, more maintainable code

---

## 💾 DATABASE USAGE

### Q7: What database did you use in this app?

**Answer:**

I used **SQLite Database** - a lightweight, embedded relational database that comes built-in with Android.

**Database Helper Class**: `DatabaseHelper.kt`
- Extends `SQLiteOpenHelper` (Android framework class)
- Manages database creation, versioning, and operations

---

### Q8: Where did you use the database in this app?

**Answer:**

The database is used throughout the application in multiple locations:

#### 1. **DatabaseHelper.kt** - Main Database Class

**Location**: `app/src/main/java/com/example/appointmentsystem/DatabaseHelper.kt`

**Database Name**: `AppointmentSystem.db`
**Database Version**: 5

**Tables Created:**

##### a) **Users Table** (`TABLE_USERS`)
```sql
CREATE TABLE users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    email TEXT UNIQUE NOT NULL,
    password TEXT NOT NULL,
    role TEXT NOT NULL,
    specialization TEXT,
    profile_picture TEXT,
    clinic_name TEXT,
    clinic_address TEXT
)
```

**Used for:**
- Storing user accounts (Doctors and Patients)
- User authentication
- Profile management

##### b) **Appointments Table** (`TABLE_APPOINTMENTS`)
```sql
CREATE TABLE appointments (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    userId INTEGER NOT NULL,
    doctorId INTEGER NOT NULL,
    date TEXT NOT NULL,
    time TEXT NOT NULL,
    status TEXT DEFAULT 'PENDING'
)
```

**Used for:**
- Storing appointment bookings
- Tracking appointment status (using `AppointmentStatus` enum)
- Managing doctor-patient appointments

##### c) **Notifications Table** (`TABLE_NOTIFICATIONS`)
```sql
CREATE TABLE notifications (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    userId INTEGER NOT NULL,
    message TEXT NOT NULL,
    timestamp TEXT NOT NULL
)
```

**Used for:**
- Storing user notifications
- Appointment reminders
- System messages

#### 2. **Database Operations Used:**

##### **Create Operations:**
- `onCreate()` - Creates all tables when database is first created
- `addUser()` - Inserts new user (Doctor or Patient)
- `addAppointment()` - Inserts new appointment
- `insertSampleDoctors()` - Inserts sample doctor data

**Location**: `DatabaseHelper.kt` lines 68-148

##### **Read Operations:**
- `getUserById()` - Retrieves user by ID
- `checkUser()` - Validates login credentials
- `getAllDoctors()` - Gets all doctors
- `getDoctorsByClinic()` - Gets doctors filtered by clinic
- `getAppointmentsByUser()` - Gets appointments for a patient
- `getAppointmentsByDoctor()` - Gets appointments for a doctor
- `getPatientsForDoctor()` - Gets all patients of a doctor
- `getDoctorStatistics()` - Gets statistics for doctor dashboard
- `getNotifications()` - Gets notifications for a user
- `getUserProfilePicture()` - Gets user profile picture

**Location**: `DatabaseHelper.kt` lines 226-737

##### **Update Operations:**
- `updateUser()` - Updates user information
- `updateAppointment()` - Updates appointment details
- `updateAppointmentStatus()` - Updates appointment status
- `updateUserProfilePicture()` - Updates profile picture

**Location**: `DatabaseHelper.kt` lines 661-779

##### **Delete Operations:**
- `cancelAppointment()` - Deletes an appointment

**Location**: `DatabaseHelper.kt` lines 647-656

#### 3. **Activities Using Database:**

##### **LoginActivity.kt**
- `dbHelper.checkUser()` - Validates login credentials
- `dbHelper.getUserById()` - Loads user session

**Location**: Lines 36, 45, 66

##### **RegisterActivity.kt**
- `dbHelper.addUser()` - Creates new user account

##### **DashboardActivity.kt**
- `dbHelper.getUserById()` - Loads current user
- `dbHelper.getDoctorStatistics()` - Gets doctor statistics

**Location**: Lines 70, 73, 220

##### **DoctorsListActivity.kt**
- `dbHelper.getAllDoctors()` - Displays list of doctors
- `dbHelper.getDoctorsByClinic()` - Filters doctors by clinic

##### **BookAppointmentActivity.kt**
- `dbHelper.addAppointment()` - Creates new appointment

##### **MyAppointmentsActivity.kt**
- `dbHelper.getAppointmentsByUser()` - Gets patient appointments
- `dbHelper.getAppointmentsByDoctor()` - Gets doctor appointments
- `dbHelper.cancelAppointment()` - Cancels appointment
- `dbHelper.updateAppointmentStatus()` - Updates status (approve/reject)
- `dbHelper.updateAppointment()` - Reschedules appointment

##### **ProfileActivity.kt**
- `dbHelper.getUserById()` - Loads user profile
- `dbHelper.updateUser()` - Updates profile information
- `dbHelper.getUserProfilePicture()` - Loads profile picture
- `dbHelper.updateUserProfilePicture()` - Updates profile picture

##### **DoctorPatientsActivity.kt**
- `dbHelper.getPatientsForDoctor()` - Lists doctor's patients

##### **NotificationActivity.kt**
- `dbHelper.getNotifications()` - Displays user notifications

---

### Q9: How does the database handle different user types (Doctor vs Patient)?

**Answer:**

The database stores both Doctors and Patients in the same `users` table, but uses the `role` column to distinguish them:

**Database Storage:**
- `role` column stores "doctor" or "patient" as TEXT
- Doctor-specific fields: `specialization`, `clinic_name`, `clinic_address`
- Patient-specific fields: None (uses only base User fields)

**Polymorphic Retrieval:**
```kotlin
fun getUserById(userId: Int): User? {
    // ... query database ...
    
    return if (role == "doctor") {
        Doctor(id, name, email, password)  // Creates Doctor object
    } else {
        Patient(id, name, email, password)  // Creates Patient object
    }
}
```

**Benefits:**
- Single table for all users (simpler schema)
- Polymorphism allows treating both as `User` type
- Easy to query and filter by role

---

## 🌐 API USAGE

### Q10: What API did you use in this app?

**Answer:**

I used **Google Maps API** through Android Intents (implicit intents).

**Note**: This is not a REST API call, but rather using Android's Intent system to interact with Google Maps application or web browser.

---

### Q11: Where did you use the API in this app?

**Answer:**

The API is used in **NearestClinicsActivity.kt**.

**Location**: `app/src/main/java/com/example/appointmentsystem/NearestClinicsActivity.kt`

#### **Implementation Details:**

##### **Method: `openClinicMap()`** (Lines 133-149)

```kotlin
private fun openClinicMap(address: String) {
    try {
        // Primary: Try to open Google Maps app
        val mapIntent = Intent(Intent.ACTION_VIEW, Uri.parse("geo:0,0?q=${Uri.encode(address)}"))
        mapIntent.setPackage("com.google.android.apps.maps")
        
        if (mapIntent.resolveActivity(packageManager) != null) {
            startActivity(mapIntent)
        } else {
            // Fallback: Open Google Maps in web browser
            val webIntent = Intent(Intent.ACTION_VIEW, 
                Uri.parse("https://www.google.com/maps/search/?api=1&query=${Uri.encode(address)}"))
            startActivity(webIntent)
        }
    } catch (e: Exception) {
        Log.e(TAG, "Error opening maps: ${e.message}", e)
        Toast.makeText(this, "Unable to open maps", Toast.LENGTH_SHORT).show()
    }
}
```

**What it does:**
1. **Primary Method**: Tries to open Google Maps Android app using `geo:` URI scheme
   - URI Format: `geo:0,0?q=[encoded address]`
   - Package: `com.google.android.apps.maps`

2. **Fallback Method**: If Google Maps app is not installed, opens Google Maps website
   - URL Format: `https://www.google.com/maps/search/?api=1&query=[encoded address]`
   - Uses `api=1` parameter (Google Maps Embed API)

**Where it's called:**
- When user clicks "View on Map" button in clinic list
- Located in `ClinicsAdapter.kt` (Lines 206-210)

```kotlin
holder.btnViewOnMap.setOnClickListener { view ->
    onMapClick(clinic)  // Calls openClinicMap() with clinic address
}
```

**Features:**
- ✅ Opens Google Maps app if installed
- ✅ Falls back to web browser if app not available
- ✅ Handles errors gracefully
- ✅ URL encoding for special characters in addresses

**Example Usage:**
- User clicks on "Central Health Clinic" → "View on Map" button
- App opens Google Maps showing: "123 Main Street, Downtown District"
- User can see location, get directions, etc.

---

### Q12: Why did you use Intent instead of direct API calls?

**Answer:**

**Benefits of using Intent approach:**

1. **No API Key Required**: Don't need to register for Google Maps API key
2. **Better User Experience**: Opens native Maps app (faster, familiar interface)
3. **Less Code**: Simpler implementation than REST API calls
4. **No Network Handling**: Android handles network requests automatically
5. **Fallback Support**: Automatically falls back to web browser if app unavailable
6. **No Dependencies**: Doesn't require additional libraries (Retrofit, OkHttp, etc.)

**Trade-offs:**
- Less control over map display
- Requires external app/website
- Can't customize map appearance as much

---

## 🔢 ENUM CONSTANTS

### Q13: What is an Enum Constant?

**Answer:**

An **Enum (Enumeration)** is a special data type that defines a set of named constants. Each constant represents a fixed value that cannot be changed.

**Characteristics:**
- ✅ Type-safe: Prevents invalid values
- ✅ Readable: Clear, meaningful names
- ✅ Compile-time checking: Errors caught during compilation
- ✅ Better than strings: Avoids typos and invalid values

**Example:**
```kotlin
enum class UserRole {
    DOCTOR,
    PATIENT
}
```

Instead of using strings like `"doctor"` or `"patient"` (which can have typos), we use `UserRole.DOCTOR` or `UserRole.PATIENT`.

---

### Q14: Where did you use Enum Constants in this app?

**Answer:**

I used **two Enum classes** in this application:

#### 1. **UserRole Enum**

**Location**: `app/src/main/java/com/example/appointmentsystem/model/Enums.kt`

```kotlin
enum class UserRole {
    DOCTOR,
    PATIENT
}
```

**Where it's used:**

##### a) **User.kt** - Base User Class
```kotlin
abstract class User(
    var id: Int = 0,
    var name: String = "",
    var email: String = "",
    var password: String = "",
    var role: UserRole  // ← Enum used here
) {
    abstract fun getRoleWelcomeMessage(): String
}
```

##### b) **Doctor.kt** - Doctor Class
```kotlin
class Doctor(...) : User(id, name, email, password, UserRole.DOCTOR) {
    // Passes UserRole.DOCTOR enum constant
}
```

##### c) **Patient.kt** - Patient Class
```kotlin
class Patient(...) : User(id, name, email, password, UserRole.PATIENT) {
    // Passes UserRole.PATIENT enum constant
}
```

**Benefits:**
- Type-safe role assignment
- Prevents typos like "doctr" or "patent"
- IDE autocomplete support
- Compile-time error checking

---

#### 2. **AppointmentStatus Enum**

**Location**: `app/src/main/java/com/example/appointmentsystem/model/Enums.kt`

```kotlin
enum class AppointmentStatus {
    PENDING,
    APPROVED,
    REJECTED,
    CANCELLED
}
```

**Where it's used:**

##### a) **Appointment.kt** - Appointment Data Class
```kotlin
data class Appointment(
    var id: Int = 0,
    var userId: Int = 0,
    var doctorId: Int = 0,
    var date: String = "",
    var time: String = "",
    var status: AppointmentStatus = AppointmentStatus.PENDING  // ← Enum used here
)
```

##### b) **DatabaseHelper.kt** - Database Operations

**When creating appointments** (Line 461):
```kotlin
fun addAppointment(appointment: Appointment): Long {
    val values = ContentValues().apply {
        put(COL_APPT_STATUS, appointment.status.name)  // Enum.name converts to string
    }
    db.insert(TABLE_APPOINTMENTS, null, values)
}
```

**When reading appointments** (Lines 588-594):
```kotlin
val statusStr = cursor.getString(cursor.getColumnIndexOrThrow(COL_APPT_STATUS))

val status = try {
    AppointmentStatus.valueOf(statusStr)  // Convert string back to enum
} catch (e: Exception) {
    AppointmentStatus.PENDING  // Default if invalid
}
```

**When updating status** (Lines 661-673):
```kotlin
fun updateAppointmentStatus(appointmentId: Int, status: AppointmentStatus): Boolean {
    val values = ContentValues().apply {
        put(COL_APPT_STATUS, status.name)  // Enum.name converts to string
    }
    db.update(TABLE_APPOINTMENTS, values, "$COL_APPT_ID = ?", arrayOf(appointmentId.toString()))
}
```

##### c) **AppointmentsAdapter.kt** - Display Status

**Displaying status with colors** (Lines 148-169):
```kotlin
holder.tvStatus.text = appointmentStatus.name.lowercase().replaceFirstChar { it.uppercase() }

when (appointmentStatus) {
    AppointmentStatus.APPROVED -> {
        holder.tvStatus.chipBackgroundColor = context.getColorStateList(R.color.status_approved)
        holder.tvStatus.setTextColor(context.getColor(R.color.status_approved_text))
    }
    AppointmentStatus.REJECTED -> {
        holder.tvStatus.chipBackgroundColor = context.getColorStateList(R.color.status_rejected)
        holder.tvStatus.setTextColor(context.getColor(R.color.status_rejected_text))
    }
    else -> { // PENDING or CANCELLED
        holder.tvStatus.chipBackgroundColor = context.getColorStateList(R.color.status_pending)
        holder.tvStatus.setTextColor(context.getColor(R.color.status_pending_text))
    }
}
```

**Checking status** (Line 181):
```kotlin
if (appointmentStatus == AppointmentStatus.PENDING) {
    holder.btnApprove?.visibility = View.VISIBLE
    holder.btnReject?.visibility = View.VISIBLE
}
```

##### d) **MyAppointmentsActivity.kt** - Status Management

**Approving appointment**:
```kotlin
dbHelper.updateAppointmentStatus(appointment.id, AppointmentStatus.APPROVED)
```

**Rejecting appointment**:
```kotlin
dbHelper.updateAppointmentStatus(appointment.id, AppointmentStatus.REJECTED)
```

**Filtering appointments**:
```kotlin
val filtered = appointments.filter { 
    it.status == AppointmentStatus.PENDING || 
    it.status == AppointmentStatus.APPROVED 
}
```

---

### Q15: Why did you use Enums instead of Strings?

**Answer:**

**Advantages of Enums:**

1. **Type Safety**:
   - ✅ `AppointmentStatus.PENDING` - Compiler checks if valid
   - ❌ `"pending"` - Can have typos: `"pendng"`, `"Pending"`, `"PENDING"`

2. **IDE Support**:
   - ✅ Autocomplete shows all available values
   - ✅ Refactoring renames all usages automatically
   - ❌ Strings require manual search/replace

3. **Compile-time Errors**:
   - ✅ `AppointmentStatus.PENDNG` → Compiler error immediately
   - ❌ `"pendng"` → Runtime error (harder to find)

4. **Better Code Readability**:
   - ✅ `if (status == AppointmentStatus.APPROVED)` - Clear intent
   - ❌ `if (status == "approved")` - Less clear

5. **Prevents Invalid Values**:
   - ✅ Can only use: PENDING, APPROVED, REJECTED, CANCELLED
   - ❌ Strings allow: `"invalid"`, `"random"`, `""`, `null`

6. **When Statement Support**:
   ```kotlin
   when (status) {
       AppointmentStatus.PENDING -> { ... }
       AppointmentStatus.APPROVED -> { ... }
       AppointmentStatus.REJECTED -> { ... }
       AppointmentStatus.CANCELLED -> { ... }
   }
   ```
   - Compiler ensures all cases are handled
   - Exhaustive checking

**Example of Problem Enums Solve:**

**Without Enum (Error-prone):**
```kotlin
// Easy to make mistakes
if (status == "pending") { ... }  // What if someone wrote "Pending"?
if (status == "approvd") { ... }  // Typo!
if (status == "PENDING") { ... }  // Case mismatch!
```

**With Enum (Safe):**
```kotlin
// Impossible to make these mistakes
if (status == AppointmentStatus.PENDING) { ... }  // ✅ Always correct
if (status == AppointmentStatus.APPROVED) { ... }  // ✅ Compiler checks
```

---

## 🎓 ADDITIONAL OOP CONCEPTS

### Q16: What other OOP patterns did you use?

**Answer:**

#### 1. **Singleton Pattern**
**Location**: `repository/UserRepository.kt`

```kotlin
object UserRepository {
    private val _currentUser = MutableLiveData<User?>()
    val currentUser: LiveData<User?> = _currentUser
    
    fun setCurrentUser(user: User?) {
        _currentUser.postValue(user)
    }
    
    fun getCurrentUser(): User? {
        return _currentUser.value
    }
    
    fun clearCurrentUser() {
        _currentUser.postValue(null)
    }
}
```

**Purpose**: Single source of truth for current logged-in user across the app
**Usage**: Accessed from multiple activities without creating multiple instances

#### 2. **Repository Pattern**
**Location**: `repository/UserRepository.kt`

- Centralizes data access logic
- Provides clean interface for user data
- Separates data source from UI

#### 3. **Adapter Pattern**
**Location**: All adapter classes (`DoctorsAdapter`, `AppointmentsAdapter`, etc.)

- Adapts data models to RecyclerView items
- Separates data from presentation
- Reusable components

#### 4. **Inheritance from Android Framework**
- All Activities extend `AppCompatActivity`
- `DatabaseHelper` extends `SQLiteOpenHelper`
- All Adapters extend `RecyclerView.Adapter`
- `BaseActivity` extends `AppCompatActivity` (custom base class)

---

### Q17: How does your app demonstrate good OOP design?

**Answer:**

**Good OOP Design Principles Demonstrated:**

1. **Single Responsibility Principle (SRP)**:
   - `DatabaseHelper` - Only handles database operations
   - `UserRepository` - Only manages user state
   - Each Activity - Handles one screen's logic
   - Each Adapter - Handles one list's display

2. **Open/Closed Principle**:
   - `User` class is open for extension (Doctor, Patient)
   - Closed for modification (don't need to change User when adding new types)

3. **Liskov Substitution Principle**:
   - `Doctor` and `Patient` can be used anywhere `User` is expected
   - Polymorphism ensures correct behavior

4. **Dependency Inversion**:
   - Activities depend on `User` abstraction, not concrete `Doctor`/`Patient`
   - Database returns `User` type, not specific implementations

5. **Encapsulation**:
   - Database details hidden in `DatabaseHelper`
   - Private constants and methods
   - Public interfaces for controlled access

6. **Code Reusability**:
   - `User` base class shared by Doctor and Patient
   - `BaseActivity` can be extended by other activities
   - Adapters follow same pattern

---

## 📝 SUMMARY

### Quick Reference:

| Concept | Location | Example |
|---------|----------|---------|
| **Abstract Class** | `model/User.kt` | `abstract class User` |
| **Inheritance** | `model/Doctor.kt`, `model/Patient.kt` | `class Doctor : User` |
| **Polymorphism** | `DashboardActivity.kt` | `user.getRoleWelcomeMessage()` |
| **Database** | `DatabaseHelper.kt` | SQLite with 3 tables |
| **API** | `NearestClinicsActivity.kt` | Google Maps Intent |
| **Enum** | `model/Enums.kt` | `UserRole`, `AppointmentStatus` |
| **Singleton** | `repository/UserRepository.kt` | `object UserRepository` |
| **Data Class** | `model/Appointment.kt` | `data class Appointment` |

---

## 🎯 EXAM TIPS

1. **Know the File Locations**: Be able to point to exact files where concepts are used
2. **Explain the Flow**: Understand how polymorphism works from database → activity → UI
3. **Code Examples**: Be ready to explain specific code snippets
4. **Why Questions**: Understand WHY you chose enums, polymorphism, etc.
5. **Database Schema**: Know your table structures and relationships
6. **API Details**: Explain how Google Maps Intent works

---

**Good Luck on Your Final Exam! 🎓**