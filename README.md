# GitApp
XLab-MHJ-129@xtremelabs.us
ppQ3cLEK

Server=tcp:dbserverakhi1234.database.windows.net,1433;Initial Catalog=appdb;Persist Security Info=False;User ID=demousr;Password={your_password};MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;

#Course
using System;
using System.Collections.Generic;
using System.Text;

namespace CourseDBFunction
{
    class Course
    {
        // This class is a representation of the structure of our data
        public int CourseID { get; set; }
        public string CourseName { get; set; }
        public decimal Rating { get; set; }
    }
}

#AddCourse

using System.Data;
using System.Data.SqlClient;
using System.IO;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Extensions.Logging;
using Newtonsoft.Json;

namespace CourseDBFunction
{
    public static class AddCourse
    {
        // This function will be used to add a course
        [FunctionName("AddCourse")]
        public static async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Function, "post", Route = null)] HttpRequest req,
            ILogger log)
        {
            // First we get the body of the POST request
            string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
            // We then use the JsonConvert class to convert the string of the request body to a Course object
            Course data = JsonConvert.DeserializeObject<Course>(requestBody);

            string _connection_string = "Server=tcp:dbserver10001.database.windows.net,1433;Initial Catalog=appdb;Persist Security Info=False;User ID=demousr;Password=Azure@123;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;";
            string _statement = "INSERT INTO Course(CourseID,CourseName,rating) VALUES(@param1,@param2,@param3)";
            SqlConnection _connection = new SqlConnection(_connection_string);
            _connection.Open();

            // Here we create a parameterized query to insert the data into the database
            using(SqlCommand _command= new SqlCommand(_statement,_connection))
            {
                _command.Parameters.Add("@param1",SqlDbType.Int).Value= data.CourseID;
                _command.Parameters.Add("@param2", SqlDbType.VarChar, 1000).Value = data.CourseName;
                _command.Parameters.Add("@param3", SqlDbType.Decimal).Value = data.Rating;
                _command.CommandType = CommandType.Text;
                _command.ExecuteNonQuery();

            }

            return new OkObjectResult("Course added");
        }
    }
}



#CourseDBFunction
using System.Collections.Generic;
using System.Data.SqlClient;
using System.IO;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Extensions.Logging;
using Newtonsoft.Json;

namespace CourseDBFunction
{
    public static class GetCourses
    {
        // This function is used to get the list of courses
        [FunctionName("GetCourses")]
        public static async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Function, "get",Route = null)] HttpRequest req,
            ILogger log)
        {
            List<Course> _lst = new List<Course>();

            string _connection_string = "Server=tcp:dbserver10001.database.windows.net,1433;Initial Catalog=appdb;Persist Security Info=False;User ID=demousr;Password=Azure@123;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;";
            string _statement = "SELECT CourseID,CourseName,rating from Course";
            // We first establish a connection to the database
            SqlConnection _connection = new SqlConnection(_connection_string);
            _connection.Open();

            SqlCommand _sqlcommand = new SqlCommand(_statement, _connection);
            // Using the SqlDataReader class , we can get all the rows from the table
            using (SqlDataReader _reader = _sqlcommand.ExecuteReader())
            {
                while (_reader.Read())
                {
                    Course _course = new Course()
                    {
                        CourseID = _reader.GetInt32(0),
                        CourseName = _reader.GetString(1),
                        Rating = _reader.GetDecimal(2)
                    };

                    _lst.Add(_course);
                }
            }
            _connection.Close();      
            // Return the HTTP status code of 200 OK and the list of courses
            return new OkObjectResult(_lst);
        }
    }
}


