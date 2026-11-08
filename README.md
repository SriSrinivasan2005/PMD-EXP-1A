
# Ex.No:1 To create a employee details fields and to display the employee details using Firebase Database in Android Studio.


## AIM:

To create and display the employee details using Firebase Database in Android Studio.

## EQUIPMENTS REQUIRED:

Android Studio(Min.required Artic Fox)

## ALGORITHM:

Step 1: Open Android Stdio and then click on File -> New -> New project.

Step 2: Then type the Application name as HelloWorld and click Next. 

Step 3: Then select the Minimum SDK as shown below and click Next.

Step 4: Then select the Empty Activity and click Next. Finally click Finish.

Step 5: Design layout in activity_main.xml.

Step 6: Display the employee details in MainActivity file.

Step 7: Save and run the application.

## PROGRAM:
```
/*
Program to print the DatabaseTable using the firebasedatabase”.
Developed by: SRI SRINIVASAN K
Registeration Number : 212224220104
*/
```

## A.Activity_Main.XML
```java
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp"
    tools:context=".MainActivity">

    <EditText
        android:id="@+id/etName"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Enter Name"
        android:inputType="textPersonName" />

    <EditText
        android:id="@+id/etAge"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Enter Age"
        android:inputType="number" />

    <EditText
        android:id="@+id/etSalary"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Enter Salary"
        android:inputType="numberDecimal" />

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:layout_marginTop="8dp">

        <Button
            android:id="@+id/btnAdd"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="Add" />

        <Button
            android:id="@+id/btnDelete"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:layout_marginStart="8dp"
            android:text="Delete" />
    </LinearLayout>

    <ListView
        android:id="@+id/listView"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:layout_marginTop="16dp" />

</LinearLayout>

```
## B. Mainctvity.java
```java
package com.example.pmd1;

import android.os.Bundle;
import android.widget.*;
import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;
import com.google.firebase.database.*;

import java.util.ArrayList;

public class MainActivity extends AppCompatActivity {

    EditText etName, etAge, etSalary;
    Button btnAdd, btnDelete;
    ListView listView;

    DatabaseReference dbRef;

    ArrayList<String> items = new ArrayList<>();
    ArrayList<String> keys = new ArrayList<>();

    ArrayAdapter<String> adapter;
    String selectedKey = null;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        etName = findViewById(R.id.etName);
        etAge = findViewById(R.id.etAge);
        etSalary = findViewById(R.id.etSalary);

        btnAdd = findViewById(R.id.btnAdd);
        btnDelete = findViewById(R.id.btnDelete);

        listView = findViewById(R.id.listView);

        dbRef = FirebaseDatabase.getInstance("https://pmd1-218e3-default-rtdb.firebaseio.com")
                .getReference("Employees");

        adapter = new ArrayAdapter<>(
                this,
                android.R.layout.simple_list_item_1,
                items
        );

        listView.setAdapter(adapter);

        // Read employee data from Firebase
        dbRef.addValueEventListener(new ValueEventListener() {

            @Override
            public void onDataChange(@NonNull DataSnapshot snapshot) {

                items.clear();
                keys.clear();

                for (DataSnapshot d : snapshot.getChildren()) {

                    Emp e = d.getValue(Emp.class);

                    if (e != null) {
                        items.add(e.name + " | " + e.age
                                + " | " + e.salary);

                        keys.add(d.getKey());
                    }
                }

                adapter.notifyDataSetChanged();
            }

            @Override
            public void onCancelled(@NonNull DatabaseError error) {
                Toast.makeText(MainActivity.this,
                        "Failed to load data",
                        Toast.LENGTH_SHORT).show();
            }
        });

        // ADD employee
        btnAdd.setOnClickListener(v -> {

            String name = etName.getText().toString().trim();
            String age = etAge.getText().toString().trim();
            String salary = etSalary.getText().toString().trim();

            if (name.isEmpty() || age.isEmpty() || salary.isEmpty()) {
                Toast.makeText(this,
                        "Enter all details",
                        Toast.LENGTH_SHORT).show();
                return;
            }

            Emp e = new Emp(name, age, salary);

            dbRef.push().setValue(e)
                    .addOnSuccessListener(unused -> {
                        Toast.makeText(this,
                                "Employee Added",
                                Toast.LENGTH_SHORT).show();
                        clearFields();
                    })
                    .addOnFailureListener(error -> Toast.makeText(this,
                            "Add Failed",
                            Toast.LENGTH_SHORT).show());
        });

        // Select employee from ListView
        listView.setOnItemClickListener((p, v, pos, id) -> {

            selectedKey = keys.get(pos);

            Toast.makeText(this,
                    "Employee Selected",
                    Toast.LENGTH_SHORT).show();
        });

        // DELETE employee
        btnDelete.setOnClickListener(v -> {

            if (selectedKey != null) {

                dbRef.child(selectedKey).removeValue()
                        .addOnSuccessListener(unused -> {
                            Toast.makeText(this,
                                    "Employee Deleted",
                                    Toast.LENGTH_SHORT).show();
                            clearFields();
                        })
                        .addOnFailureListener(error -> Toast.makeText(this,
                                "Delete Failed",
                                Toast.LENGTH_SHORT).show());

            } else {
                Toast.makeText(this,
                        "Select an employee first",
                        Toast.LENGTH_SHORT).show();
            }
        });
    }

    private void clearFields() {
        etName.setText("");
        etAge.setText("");
        etSalary.setText("");
        selectedKey = null;
    }
}

```
## C. Emp.java
```java
package com.example.pmd1;

public class Emp {
    public String id;
    public String name;
    public String age;
    public String salary;

    public Emp() {
        // Required for Firebase
    }

    public Emp(String name, String age, String salary) {
        this.name = name;
        this.age = age;
        this.salary = salary;
    }

    public Emp(String id, String name, String age, String salary) {
        this.id = id;
        this.name = name;
        this.age = age;
        this.salary = salary;
    }
}

```
## OUTPUT

<img width="1920" height="1200" alt="Screenshot (95)" src="https://github.com/user-attachments/assets/148ac18b-99f8-4d91-b070-07bd53d64bcd" />

<img width="1920" height="1200" alt="Screenshot (97)" src="https://github.com/user-attachments/assets/a1bc0033-0dcb-4eee-be37-ccfcb7c2bcc1" />

<img width="1920" height="1200" alt="Screenshot (96)" src="https://github.com/user-attachments/assets/31204c7f-ffce-4d79-bad3-e6e90ddab39d" />



## RESULT
Thus a Simple Android Application create a firebase database and to display the employee details using Firbase Real Time Database in Android Studio is developed and executed successfully.
