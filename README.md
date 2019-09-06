# flutter_sqlite_crud
 How to perform CRUD operation in SQlite Database in Flutter

## Introduction:
 In this article we will learn how to implement SQlite Database in Flutter Application. SQlite is used to store the data in relational tables in local device. The Database is persistent in nature until you do not remove database from the device. SQlite database is mostly used to store data in offline mode. SQFlite plugin is used to implement SQlite database in Flutter Application so let’s take Student example to perform CRUD Operation in Flutter.

## Output:

![Flutter SQlite CRUD Operation](https://raw.githubusercontent.com/myvsparth/flutter_sqlite_crud/master/screenshots/1.png)

## Plugin Required:
 sqflite: ^1.1.6+4
 path_provider: ^1.3.0

## Steps:
 Step 1: First and basic step to create new application in flutter. If you are a beginner in flutter then you can check my blog Create a first app in Flutter. I have created an app named as “flutter_sqlite_crud”.

 Step 2: Now, We will configure two plugins in pubspec.yaml file sqflite and path_provider.
```
dependencies:
 flutter:
   sdk: flutter
 cupertino_icons: ^0.1.2
 sqflite: ^1.1.6+4
 path_provider: ^1.3.0
```

 Step 3: Now, we will implement modal for student data structure. For that we create student_model.dart file and create student class with id and name properties and provide mapping for sqlite database. Following is the programming implementation of that.
```
class Student {
 int id;
 String name;
 Student(this.id, this.name);
 
 Map<String, dynamic> toMap() {
   var map = <String, dynamic>{
     'id': id,
     'name': name,
   };
   return map;
 }
 
 Student.fromMap(Map<String, dynamic> map) {
   id = map['id'];
   name = map['name'];
 }
}
```

 Step 4: Now, we will implement Sqlite Database CRUD Operation implementation. For that create a file named as  db_helper.dart and initialize the database. Then create insert, delete, update and select functions for CRUD operation. Following is the programming implementation of that.
```
import 'package:flutter_sqlite_crud/student_model.dart';
import 'package:sqflite/sqflite.dart';
import 'dart:io' as io;
import 'package:path/path.dart';
import 'package:path_provider/path_provider.dart';
 
class DBHelper {
 static Database _db;
 Future<Database> get db async {
   if (_db != null) {
     return _db;
   }
   _db = await initDatabase();
   return _db;
 }
 
 initDatabase() async {
   io.Directory documentDirectory = await getApplicationDocumentsDirectory();
   String path = join(documentDirectory.path, 'student.db');
   var db = await openDatabase(path, version: 1, onCreate: _onCreate);
   return db;
 }
 
 _onCreate(Database db, int version) async {
   await db
       .execute('CREATE TABLE student (id INTEGER PRIMARY KEY, name TEXT)');
 }
 
 Future<Student> add(Student student) async {
   var dbClient = await db;
   student.id = await dbClient.insert('student', student.toMap());
   return student;
 }
 
 Future<List<Student>> getStudents() async {
   var dbClient = await db;
   List<Map> maps = await dbClient.query('student', columns: ['id', 'name']);
   List<Student> students = [];
   if (maps.length > 0) {
     for (int i = 0; i < maps.length; i++) {
       students.add(Student.fromMap(maps[i]));
     }
   }
   return students;
 }
 
 Future<int> delete(int id) async {
   var dbClient = await db;
   return await dbClient.delete(
     'student',
     where: 'id = ?',
     whereArgs: [id],
   );
 }
 
 Future<int> update(Student student) async {
   var dbClient = await db;
   return await dbClient.update(
     'student',
     student.toMap(),
     where: 'id = ?',
     whereArgs: [student.id],
   );
 }
 
 Future close() async {
   var dbClient = await db;
   dbClient.close();
 }
}
```

 Step 5: Now, We will implement UI side by creating form to input student detail and data table to show student records. Following is the programming implementation of that.
```
import 'package:flutter/material.dart';
import 'package:flutter_sqlite_crud/db_helper.dart';
import 'package:flutter_sqlite_crud/student_model.dart';
 
void main() => runApp(MyApp());
 
class MyApp extends StatelessWidget {
 @override
 Widget build(BuildContext context) {
   return MaterialApp(
     theme: ThemeData(
       primarySwatch: Colors.purple,
     ),
     home: StudentPage(),
   );
 }
}
 
class StudentPage extends StatefulWidget {
 @override
 _StudentPageState createState() => _StudentPageState();
}
 
class _StudentPageState extends State<StudentPage> {
 final GlobalKey<FormState> _formStateKey = GlobalKey<FormState>();
 Future<List<Student>> students;
 String _studentName;
 bool isUpdate = false;
 int studentIdForUpdate;
 DBHelper dbHelper;
 final _studentNameController = TextEditingController();
 
 @override
 void initState() {
   super.initState();
   dbHelper = DBHelper();
   refreshStudentList();
 }
 
 refreshStudentList() {
   setState(() {
     students = dbHelper.getStudents();
   });
 }
 
 @override
 Widget build(BuildContext context) {
   return Scaffold(
     appBar: AppBar(
       title: Text('SQLite CRUD in Flutter'),
     ),
     body: Column(
       children: <Widget>[
         Form(
           key: _formStateKey,
           autovalidate: true,
           child: Column(
             children: <Widget>[
               Padding(
                 padding: EdgeInsets.only(left: 10, right: 10, bottom: 10),
                 child: TextFormField(
                   validator: (value) {
                     if (value.isEmpty) {
                       return 'Please Enter Student Name';
                     }
                     if (value.trim() == "")
                       return "Only Space is Not Valid!!!";
                     return null;
                   },
                   onSaved: (value) {
                     _studentName = value;
                   },
                   controller: _studentNameController,
                   decoration: InputDecoration(
                       focusedBorder: new UnderlineInputBorder(
                           borderSide: new BorderSide(
                               color: Colors.purple,
                               width: 2,
                               style: BorderStyle.solid)),
                       // hintText: "Student Name",
                       labelText: "Student Name",
                       icon: Icon(
                         Icons.business_center,
                         color: Colors.purple,
                       ),
                       fillColor: Colors.white,
                       labelStyle: TextStyle(
                         color: Colors.purple,
                       )),
                 ),
               ),
             ],
           ),
         ),
         Row(
           mainAxisAlignment: MainAxisAlignment.center,
           children: <Widget>[
             RaisedButton(
               color: Colors.purple,
               child: Text(
                 (isUpdate ? 'UPDATE' : 'ADD'),
                 style: TextStyle(color: Colors.white),
               ),
               onPressed: () {
                 if (isUpdate) {
                   if (_formStateKey.currentState.validate()) {
                     _formStateKey.currentState.save();
                     dbHelper
                         .update(Student(studentIdForUpdate, _studentName))
                         .then((data) {
                       setState(() {
                         isUpdate = false;
                       });
                     });
                   }
                 } else {
                   if (_formStateKey.currentState.validate()) {
                     _formStateKey.currentState.save();
                     dbHelper.add(Student(null, _studentName));
                   }
                 }
                 _studentNameController.text = '';
                 refreshStudentList();
               },
             ),
             Padding(
               padding: EdgeInsets.all(10),
             ),
             RaisedButton(
               color: Colors.red,
               child: Text(
                 (isUpdate ? 'CANCEL UPDATE' : 'CLEAR'),
                 style: TextStyle(color: Colors.white),
               ),
               onPressed: () {
                 _studentNameController.text = '';
                 setState(() {
                   isUpdate = false;
                   studentIdForUpdate = null;
                 });
               },
             ),
           ],
         ),
         const Divider(
           height: 5.0,
         ),
         Expanded(
           child: FutureBuilder(
             future: students,
             builder: (context, snapshot) {
               if (snapshot.hasData) {
                 return generateList(snapshot.data);
               }
               if (snapshot.data == null || snapshot.data.length == 0) {
                 return Text('No Data Found');
               }
               return CircularProgressIndicator();
             },
           ),
         ),
       ],
     ),
   );
 }
 
 SingleChildScrollView generateList(List<Student> students) {
   return SingleChildScrollView(
     scrollDirection: Axis.vertical,
     child: SizedBox(
       width: MediaQuery.of(context).size.width,
       child: DataTable(
         columns: [
           DataColumn(
             label: Text('NAME'),
           ),
           DataColumn(
             label: Text('DELETE'),
           )
         ],
         rows: students
             .map(
               (student) => DataRow(
                 cells: [
                   DataCell(
                     Text(student.name),
                     onTap: () {
                       setState(() {
                         isUpdate = true;
                         studentIdForUpdate = student.id;
                       });
                       _studentNameController.text = student.name;
                     },
                   ),
                   DataCell(
                     IconButton(
                       icon: Icon(Icons.delete),
                       onPressed: () {
                         dbHelper.delete(student.id);
                         refreshStudentList();
                       },
                     ),
                   )
                 ],
               ),
             )
             .toList(),
       ),
     ),
   );
 }
}
```
 Hurrey…. Run the app and Test It on emulator/simulator or device :)))

## Conclusion:
 We have learnt how to implement SQLite Database in Flutter and Perform CRUD operation. 

> Git Repo: https://github.com/myvsparth/flutter_sqlite_crud
## Related to Tags: Flutter, SQLite Database, SQFlite Plugin, CRUD Operation