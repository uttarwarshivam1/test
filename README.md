# test
          //Database
create database college;
use college;
create table student (id integer primary key auto_increment, name varchar(50), course varchar(50), );

insert into student values(50, "Suraj", "Java");
insert into student(name, course, purchasedate) values("Rohit", "Python");

-------------------------------------------CMD BACKEND----------------------------------------------------------------
1) create folder Backend and go into that folder then opne the CMD over there

2) dotnet new sln

3) dotnet new webapi -o myapp
   cd myapp
   dotnet add package MySql.data
   dotnet add package MicrosoftAspCore.Cors

4) go to vs code
   create new folder Model in Backend Folder and create Student.cs (POCO class) file in model

5) create new folder DAL and create File StudentData.cs in DAL file

6) In controller create StudentController.cs file
  

------------------------------------CMD FRONTEND-----------------------------

create new folder frontend and open CMD over there

npm install-create-react-app -g
npm create-react-app myapp

cd myapp
npm install axios
npm start
                *******************END OF ALL COMMANDS***********************************
--------------------------------------------------------------------------------------------------------
                     *********BACKEND CODE*******************
[MODEL ==> Student.cs]
namespace Model;

public class Student
{
    public int id{get;set;}

    public string? name{get;set;}="";
    
    public  string? course{get;set;}="";

}
     
------------------------StudentData.cs-------------------------------------------------------------------------
[DAL ==> StudentData.cs]

namespace DAL;
using System.Data;
using Model;
using MySql.Data.MySqlClient;

public class StudentData
{
    public static string conString=@"server=localhost;port=3306;user=root; password=Shubham@0154;database=college";
//------------GetAllStudents------
    public static List<Student>GetAllStudents()
    {
        List<Student> allNotes = new List<Student>();

        MySqlConnection con = new MySqlConnection(conString);

        try
        {
            String query ="select * from student";
            DataSet ds = new DataSet();
            MySqlCommand cmd =  new MySqlCommand(query,con);
            MySqlDataAdapter da = new MySqlDataAdapter(cmd);
            da.Fill(ds);

            DataTable dt = ds.Tables[0];
            DataRowCollection rows = dt.Rows;
            foreach (DataRow row in rows)
            {
                Student student = new Student
                {
                    id=int.Parse(row["id"].ToString()),
                    name=row["name"].ToString(),
                    course=row["course"].ToString()
                };
                allNotes.Add(student);            
            }
        }
        catch(Exception e)
        {
            Console.WriteLine(e.Message);
        }
        return allNotes;
    } 
//-----InsertNewStudent-----
    public static void InsertNewStudent(Student student)
    {
        MySqlConnection con = new MySqlConnection(conString);

        try 
        {
            con.Open();
            string query =$"insert into student(name,course) values('{student.name}','{student.course}')";
            MySqlCommand command = new MySqlCommand(query,con);
            command.ExecuteNonQuery();
            con.Close();
        }
        catch (Exception e)
        {

            Console.WriteLine(e.Message);
        }
        finally
        {
            con.Close();
        }
    }
//----DeleteStudent----
    public static void DeleteStudent(int id)
    {
        MySqlConnection con = new MySqlConnection(conString);

        try{
            con.Open();                                         //#opne the connection
            string query ="delete from student where id =" + id; //# write the query
            MySqlCommand command = new MySqlCommand(query,con);  //#add query to Database
            command.ExecuteNonQuery(); 
            con.Close();  //# Cloing the connection
        }
        catch (Exception e)
        {
            Console.WriteLine(e.Message);
        }
        finally
        {
            con.Close();
        }
    }

}

-------------------------StudentController.cs------------------------------------------------------------------------------
//[Controller ==> StudentController.cs]

namespace webapi.Controllers;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Cors;
using DAL;
using Model;


[ApiController]
[Route("api/[controller]")]
public class StudentController : ControllerBase
{

    private readonly ILogger<StudentController> _logger;

    public StudentController(ILogger<StudentController> logger)
    {
        _logger = logger;
    }

    [HttpGet]
    [EnableCors()]
    public IEnumerable<Student> GetAllStudents()
    {
       List<Student> students=StudentData.GetAllStudents();

       return students;     
    }


//------

[HttpPost]
[EnableCors()]
public IActionResult InsertNewStudent(Student student)
{
    StudentData.InsertNewStudent(student);
    return Ok(new { message ="User created"});
}

//--
[Route("{id}")]
[HttpDelete]
[EnableCors()]
public ActionResult<Student> DeleteStudent(int id)
{
    StudentData.DeleteStudent(id);
    return Ok(new { message = "Student delete"});
}

}

------------------------Program.cs-----------------------------------------------------
//[Program.cs]

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddCors();
// Add services to the container.

builder.Services.AddControllers();
// Learn more about configuring Swagger/OpenAPI at https://aka.ms/aspnetcore/swashbuckle
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();

app.UseCors(builder => {
    builder.AllowAnyOrigin()
        .AllowAnyMethod()
        .AllowAnyHeader();
});

app.UseAuthorization();

app.MapControllers();

app.Run();

----------------------------------------------------------------------------------
                    ************FRONTEND CODE*****************

//[App.js]

import logo from './logo.svg';
import './App.css';
import DisplayStudents from './Components/DisplayStudents';
import InsertStudent from './Components/InsertStudent';
import DeletStudent from './Components/DeleteStudent';
function App() {
  return (
    <div>
      <DisplayStudents></DisplayStudents>
      <InsertStudent></InsertStudent>
      <DeletStudent></DeletStudent>
    </div>
  );
}

export default App;

------------------DisplayStudent.js------------------------------------------

//[Components ==> DisplayStudent.js]

import {useEffect,useState} from "react";
import axios from 'axios';

 const  DisplayStudents =(props) => {
     const [studarr,setstudarr] = useState([]);
     useEffect(
         () => {
             axios.get('http://localhost:5196/api/student')
             .then(respone =>{setstudarr(respone.data)})
         }
     )
//-----
     var tablerows = studarr.map(obj =>{
         return(
             <tr>
             <td>{obj.id}</td>
             <td>{obj.name}</td>
             <td>{obj.course}</td>
             </tr>
         );
     });
//-----
     return (
         <>
         <br></br>
         <table id="studentTable" border="2px" bgcolor="cyan" >
         <tr>
         <td> Student Id </td>
         <td> Name </td>
         <td> Course</td>
         </tr>
         {tablerows}
         </table>
         </>
         );

 }

export default DisplayStudents;

------------------DeleteStudent.js--------------------------------------------

//[Components ==> DeleteStudent.js]

import { useState } from "react";
import axios from 'axios';

const DeleteData = () => {
    const [studData, setstudData] = useState({id:""});

    const deleteStudent = (event) => {
        event.preventDefault();
        axios.delete(`http://localhost:5196/api/student/${studData.id}`);
    }

    const handleChange =(event) => {
        const {name,value}=event.target
        setstudData({...studData,[name]:value})
    }

    return (
        <>
        <br/><br/>
        <h4> Enter id to be deleted</h4>
        <form method="GET" onSubmit={deleteStudent}>
            <input type="text" name="id" onChange={handleChange} placeholder="id"/>
            <input type="Submit" value="Delete"/>
        </form>
        </>
    );
}
export default DeleteData;

----------------------------InsertStudent.js-----------------------------------

//[Components ==> InsertStudent.js]

import { useState} from "react";
import axios from 'axios';

const Insertdata = (props) => {
    const [studData,setstudData]= useState({name:"",course:""});

    const savedata =(event) => {
        event.preventDefault();
        axios.post('http://localhost:5196/api/student',studData);
    }

    const handleChange=(event)=>{
        const {name,value} =event.target
        setstudData({...studData,[name]:value})
    }


    return(
        <>
        <br/><br/>
        <h4>Add new Student</h4>
        <form method="POST" onSubmit={savedata}>
            <input type="text" name="name" onChange={handleChange} placeholder="StudentName"/>
            <input type="text" name="course" onChange={handleChange} placeholder="course"/>
            <input type="Submit"/>
        </form>
        </>
    );
}
export default Insertdata;

------------------------------END------------------------------------------------------------------------------------------------
