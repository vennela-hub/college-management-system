# college-management-system
CREATE DATABASE college_admission;

USE college_admission;

-- Students Table
CREATE TABLE Students (
    student_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100) UNIQUE,
    marks INT
);

-- Courses Table
CREATE TABLE Courses (
    course_id INT AUTO_INCREMENT PRIMARY KEY,
    course_name VARCHAR(100),
    cutoff_marks INT
);

-- Applications Table
CREATE TABLE Applications (
    app_id INT AUTO_INCREMENT PRIMARY KEY,
    student_id INT,
    course_id INT,
    status VARCHAR(20) DEFAULT 'Pending',
    FOREIGN KEY (student_id) REFERENCES Students(student_id),
    FOREIGN KEY (course_id) REFERENCES Courses(course_id)
);
import java.sql.*;
import java.util.*;
import java.io.FileWriter;

public class CollegeAdmissionSystem {
    private static final String URL = "jdbc:mysql://localhost:3306/college_admission";
    private static final String USER = "root";  // change your MySQL username
    private static final String PASSWORD = "password"; // change your MySQL password

    private Connection conn;

    public CollegeAdmissionSystem() throws Exception {
        conn = DriverManager.getConnection(URL, USER, PASSWORD);
    }

    // Register student
    public void registerStudent(String name, String email, int marks) throws Exception {
        String sql = "INSERT INTO Students(name, email, marks) VALUES(?,?,?)";
        try (PreparedStatement ps = conn.prepareStatement(sql)) {
            ps.setString(1, name);
            ps.setString(2, email);
            ps.setInt(3, marks);
            ps.executeUpdate();
            System.out.println("Student registered successfully.");
        }
    }

    // Apply for course
    public void applyCourse(int studentId, int courseId) throws Exception {
        String sql = "INSERT INTO Applications(student_id, course_id) VALUES(?,?)";
        try (PreparedStatement ps = conn.prepareStatement(sql)) {
            ps.setInt(1, studentId);
            ps.setInt(2, courseId);
            ps.executeUpdate();
            System.out.println("Application submitted.");
        }
    }

    // Admin: process applications
    public void processApplications() throws Exception {
        String sql = "SELECT a.app_id, s.name, s.marks, c.course_name, c.cutoff_marks " +
                     "FROM Applications a " +
                     "JOIN Students s ON a.student_id = s.student_id " +
                     "JOIN Courses c ON a.course_id = c.course_id " +
                     "WHERE a.status = 'Pending'";

        try (Statement st = conn.createStatement(); ResultSet rs = st.executeQuery(sql)) {
            while (rs.next()) {
                int appId = rs.getInt("app_id");
                String studentName = rs.getString("name");
                int marks = rs.getInt("marks");
                String course = rs.getString("course_name");
                int cutoff = rs.getInt("cutoff_marks");

                String decision = (marks >= cutoff) ? "Approved" : "Rejected";

                // Update status
                try (PreparedStatement ps = conn.prepareStatement("UPDATE Applications SET status=? WHERE app_id=?")) {
                    ps.setString(1, decision);
                    ps.setInt(2, appId);
                    ps.executeUpdate();
                }

                System.out.println("Application #" + appId + " (" + studentName + " → " + course + ") : " + decision);
            }
        }
    }

    // Export admission list to CSV
    public void exportAdmissionList() throws Exception {
        String sql = "SELECT s.name, s.email, s.marks, c.course_name, a.status " +
                     "FROM Applications a " +
                     "JOIN Students s ON a.student_id = s.student_id " +
                     "JOIN Courses c ON a.course_id = c.course_id";

        try (Statement st = conn.createStatement(); ResultSet rs = st.executeQuery(sql)) {
            FileWriter csv = new FileWriter("admission_list.csv");
            csv.write("Name,Email,Marks,Course,Status\n");

            while (rs.next()) {
                csv.write(rs.getString("name") + "," +
                          rs.getString("email") + "," +
                          rs.getInt("marks") + "," +
                          rs.getString("course_name") + "," +
                          rs.getString("status") + "\n");
            }
            csv.close();
            System.out.println("Admission list exported to admission_list.csv");
        }
    }

    // Console Menu
    public static void main(String[] args) {
        try {
            CollegeAdmissionSystem system = new CollegeAdmissionSystem();
            Scanner sc = new Scanner(System.in);

            while (true) {
                System.out.println("\n===== College Admission Menu =====");
                System.out.println("1. Register Student");
                System.out.println("2. Apply for Course");
                System.out.println("3. Process Applications (Admin)");
                System.out.println("4. Export Admission List");
                System.out.println("5. Exit");
                System.out.print("Enter choice: ");
                int choice = sc.nextInt();
                sc.nextLine();

                switch (choice) {
                    case 1:
                        System.out.print("Enter name: ");
                        String name = sc.nextLine();
                        System.out.print("Enter email: ");
                        String email = sc.nextLine();
                        System.out.print("Enter marks: ");
                        int marks = sc.nextInt();
                        system.registerStudent(name, email, marks);
                        break;

                    case 2:
                        System.out.print("Enter student ID: ");
                        int sid = sc.nextInt();
                        System.out.print("Enter course ID: ");
                        int cid = sc.nextInt();
                        system.applyCourse(sid, cid);
                        break;

                    case 3:
                        system.processApplications();
                        break;

                    case 4:
                        system.exportAdmissionList();
                        break;

                    case 5:
                        System.out.println("Goodbye!");
                        return;

                    default:
                        System.out.println("Invalid choice.");
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
