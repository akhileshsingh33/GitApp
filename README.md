# GitApp
XLab-EQA-743@xtremelabs.us
1NQlqCyI
#Course
public int CourseID { get; set; }
public string CourseName { get; set; }
public decimal Rating { get; set; }

#CourseService
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using System.Data.SqlClient;
using Sqlapp.Models;

namespace Sqlapp.Services
{
    public class CourseService
    {
        // Ensure to change the below variables to reflect the connection details for your database
        private static string db_source = "dbserver10001.database.windows.net";
        private static string db_user = "demousr";
        private static string db_password = "Azure@123";
        private static string db_database = "appdb";

        private SqlConnection GetConnection()
        {
            // Here we are creating the SQL connection
            var _builder = new SqlConnectionStringBuilder();
            _builder.DataSource = db_source;
            _builder.UserID = db_user;
            _builder.Password = db_password;
            _builder.InitialCatalog = db_database;
            return new SqlConnection(_builder.ConnectionString);
        }

        public IEnumerable<Course> GetCourses()
        {
            List<Course> _lst = new List<Course>();
            string _statement = "SELECT CourseID,CourseName,rating from Course";
            SqlConnection _connection = GetConnection();
            // Let's open the connection
            _connection.Open();
            // We then construct the statement of getting the data from the Course table
            SqlCommand _sqlcommand = new SqlCommand(_statement, _connection);
            // Using the SqlDataReader class , we will read all the data from the Course table
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
            return _lst;
        }

    }
    }

    
    # CourseController
    
    using Microsoft.AspNetCore.Mvc;
using Sqlapp.Models;
using Sqlapp.Services;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace Sqlapp.Controllers
{
    public class CourseController : Controller
    {
        private readonly CourseService _course_service;

        public CourseController(CourseService _svc)
        {
            _course_service = _svc;
        }

        // The Index method is used to get a list of courses and return it to the view
        public IActionResult Index()
        {
            IEnumerable<Course> _course_list = _course_service.GetCourses();
            return View(_course_list);
        }
    }
}

# Index.cshtml

@model IEnumerable<Sqlapp.Models.Course>
@{
    ViewData["Title"] = "Home page";
}
<head>


    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css" integrity="sha384-ggOyR0iXCbMQv3Xipma34MD+dH/1fQ784/j6cY/iJTQUOhcWr7x9JvoRxT2MZw1T" crossorigin="anonymous">
</head>

<div class="text-center">
    <h1 class="display-4">This is a list of services of courses</h1>

</div>

<div>
    <table class="table" table-dark">
        <thead>
            <tr>
                <th scope="col">Course ID</th>
                <th scope="col">Course Name</th>
                <th scope="col">Rating</th>                
            </tr>
        </thead>
        <tbody>
            @foreach (var course in Model)
            {
        <tr>
            <th scope="row">@course.CourseID</th>
            <td>@course.CourseName</td>
            <td>@course.Rating</td>
        </tr>
            }            
        </tbody>
    </table>
</div>

# Startup
 public void ConfigureServices(IServiceCollection services)
        {
            // Ensure to add the services
            services.AddMvc();
            services.AddTransient<CourseService>();
        }
                           
                            app.UseRouting();

            // Ensure to map the controllers accordingly
            app.UseEndpoints(endpoints =>
            {
                endpoints.MapControllerRoute(
                    name: "default",
                    pattern: "{controller=Course}/{action=Index}/{id?}");
            });
