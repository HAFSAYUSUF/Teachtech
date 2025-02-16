# Teachtech
package com.example.teachtrack

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.background
import androidx.compose.foundation.clickable
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.foundation.text.BasicTextField
import androidx.compose.foundation.text.KeyboardActions
import androidx.compose.foundation.text.KeyboardOptions
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.text.input.ImeAction
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp
import androidx.compose.ui.text.TextStyle
import androidx.compose.ui.tooling.preview.Preview

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            TeachTrackApp()
        }
    }
}

@Composable
fun TeachTrackApp() {
    var isLoggedIn by remember { mutableStateOf(false) }

    if (isLoggedIn) {
        SubjectSelectionScreen(onLogout = { isLoggedIn = false })
    } else {
        LoginScreen(onLogin = { isLoggedIn = true })
    }
}

@Composable
fun LoginScreen(onLogin: () -> Unit) {
    var username by remember { mutableStateOf("") }
    var password by remember { mutableStateOf("") }

    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(32.dp),
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Text("Login", fontSize = 32.sp, fontWeight = FontWeight.Bold)
        Spacer(modifier = Modifier.height(32.dp))

        BasicTextField(
            value = username,
            onValueChange = { username = it },
            keyboardOptions = KeyboardOptions.Default.copy(imeAction = ImeAction.Next),
            keyboardActions = KeyboardActions(onNext = {}),
            modifier = Modifier
                .fillMaxWidth()
                .background(Color.Gray.copy(alpha = 0.1f), shape = MaterialTheme.shapes.small)
                .padding(16.dp),
            textStyle = TextStyle(fontSize = 18.sp)
        )
        Spacer(modifier = Modifier.height(16.dp))

        BasicTextField(
            value = password,
            onValueChange = { password = it },
            keyboardOptions = KeyboardOptions.Default.copy(imeAction = ImeAction.Done),
            keyboardActions = KeyboardActions(onDone = { onLogin() }),
            modifier = Modifier
                .fillMaxWidth()
                .background(Color.Gray.copy(alpha = 0.1f), shape = MaterialTheme.shapes.small)
                .padding(16.dp),
            textStyle = TextStyle(fontSize = 18.sp)
        )
        Spacer(modifier = Modifier.height(32.dp))

        Button(onClick = onLogin, modifier = Modifier.fillMaxWidth()) {
            Text("Login")
        }
    }
}

@Composable
fun SubjectSelectionScreen(onLogout: () -> Unit) {
    var showReport by remember { mutableStateOf(false) }
    var subjects = listOf("Math", "Science", "English")
    var students = listOf("Alice", "Bob", "Charlie", "David", "Emma")
    var selectedSubject by remember { mutableStateOf<String?>(null) }

    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp)
    ) {
        Text("Select an Option", fontSize = 24.sp, fontWeight = FontWeight.Bold)
        Spacer(modifier = Modifier.height(20.dp))

        Button(onClick = { showReport = false }) {
            Text("Select Subject")
        }
        Spacer(modifier = Modifier.height(16.dp))

        Button(onClick = { showReport = true }) {
            Text("View Report")
        }

        Spacer(modifier = Modifier.height(32.dp))

        if (showReport) {
            ReportScreen(students)
        } else {
            SubjectScreen(subjects, onSubjectSelected = { selectedSubject = it }, onLogout)
        }

        selectedSubject?.let {
            // Show attendance for the selected subject
            AttendanceScreen(selectedSubject = it)
        }
    }
}

@Composable
fun SubjectScreen(subjects: List<String>, onSubjectSelected: (String) -> Unit, onLogout: () -> Unit) {
    LazyColumn {
        items(subjects) { subject ->
            SubjectRow(subject = subject, onSubjectSelected)
        }
    }
}

@Composable
fun SubjectRow(subject: String, onSubjectSelected: (String) -> Unit) {
    Row(
        modifier = Modifier
            .fillMaxWidth()
            .padding(8.dp)
            .background(Color.LightGray)
            .clickable { onSubjectSelected(subject) },
        horizontalArrangement = Arrangement.SpaceBetween,
        verticalAlignment = Alignment.CenterVertically
    ) {
        Text(subject, fontSize = 18.sp, modifier = Modifier.padding(16.dp))
    }
}

@Composable
fun ReportScreen(students: List<String>) {
    LazyColumn {
        items(students) { student ->
            ReportRow(student)
        }
    }
}

@Composable
fun ReportRow(student: String) {
    Row(
        modifier = Modifier
            .fillMaxWidth()
            .padding(8.dp)
            .background(Color.White)
    ) {
        Text(student, fontSize = 18.sp, modifier = Modifier.padding(16.dp))
        Spacer(modifier = Modifier.weight(1f))
        Text("Attendance: 85%", fontSize = 16.sp, modifier = Modifier.padding(16.dp))
    }
}

@Composable
fun AttendanceScreen(selectedSubject: String) {
    var students = listOf("Alice", "Bob", "Charlie", "David", "Emma")
    var attendanceMap = remember { mutableStateMapOf<String, Boolean>() }

    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp)
    ) {
        Text("Mark Attendance for $selectedSubject", fontSize = 24.sp, fontWeight = FontWeight.Bold)
        Spacer(modifier = Modifier.height(20.dp))

        LazyColumn {
            items(students) { student ->
                AttendanceRow(student = student, attendanceMap)
            }
        }

        Spacer(modifier = Modifier.height(20.dp))
        Button(onClick = { exportAttendance(attendanceMap) }) {
            Text("Export Attendance")
        }
    }
}

@Composable
fun AttendanceRow(student: String, attendanceMap: MutableMap<String, Boolean>) {
    Row(
        modifier = Modifier
            .fillMaxWidth()
            .padding(8.dp)
            .background(Color.LightGray)
            .clickable {
                attendanceMap[student] = !(attendanceMap[student] ?: false)
            },
        horizontalArrangement = Arrangement.SpaceBetween,
        verticalAlignment = Alignment.CenterVertically
    ) {
        Text(student, fontSize = 18.sp, modifier = Modifier.padding(16.dp))
        Spacer(modifier = Modifier.weight(1f))
        Text(
            if (attendanceMap[student] == true) "Present" else "Absent",
            fontSize = 16.sp,
            modifier = Modifier.padding(16.dp)
        )
    }
}

fun exportAttendance(attendanceMap: Map<String, Boolean>) {
    // Handle export logic here
}

@Composable
@Preview(showBackground = true)
fun DefaultPreview() {
    TeachTrackApp()
}


