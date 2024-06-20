package com.gniot.crs.dao;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Timestamp;

import com.gniot.crs.bean.*;
import com.gniot.crs.constant.SQLConstant;
import com.gniot.crs.exception.*;
import com.gniot.crs.utils.DBUtils;

import java.util.*;

/**
 * Implementation of ProfessorDAOInterface. This class provides the SQL
 * implementation for managing grades and viewing enrolled students in the
 * course registration system.
 */
public class ProfessorDAOImpl implements ProfessorDAOInterface {
	final String GREEN = "\u001B[32m"; // ANSI escape code for green text
	final String RED = "\u001B[31m"; // ANSI escape code for red text

	final String YELLOW = "\u001B[33m";
	final String RESET = "\u001B[0m";
	UserDAOImpl userDAO = new UserDAOImpl();
	/**
	 * Adds a grade for a student in a specific course. This method includes SQL to
	 * insert or update the grade record in the database.
	 *
	 * @param studentId the ID of the student to whom the grade is assigned
	 * @param courseId  the ID of the course for which the grade is assigned
	 * @param grade     the grade to be assigned
	 * @return boolean indicating the success or failure of the operation
	 */
    public void addGrade(int studentId, int courseId, String grade) {
        try (Connection conn = DBUtils.getConnection()) {

            // Check if the course is taught by the professor
            if (!isCourseTaughtByProfessor(conn, studentId, courseId, currentProfessorId())) {
                throw new UnauthorizedGradeException(RED + "Error: You are not authorized to add grades for this student in this course." + RESET);
            }

            // Check if the grade record exists
            try (PreparedStatement checkStmt = conn.prepareStatement(SQLConstant.FETCH_ADD_GRADE)) {
                checkStmt.setInt(1, studentId);
                checkStmt.setInt(2, courseId);
                try (ResultSet rs = checkStmt.executeQuery()) {
                    if (rs.next()) {
                        // Update existing grade
                        try (PreparedStatement updateStmt = conn.prepareStatement(SQLConstant.UPDATE_ADD_GRADE)) {
                            updateStmt.setString(1, grade);
                            updateStmt.setTimestamp(2, new Timestamp(System.currentTimeMillis()));
                            updateStmt.setInt(3, studentId);
                            updateStmt.setInt(4, courseId);
                            updateStmt.executeUpdate();
                            System.out.println(GREEN + "Grade updated successfully for student " + studentId + " in course " + courseId + RESET);
                        }
                    } else {
                        // Insert new grade (no need to fetch course details, they're already provided)
                        try (PreparedStatement insertStmt = conn.prepareStatement(SQLConstant.INSERT_ADD_GRADE)) {
                            insertStmt.setInt(1, studentId);
                            insertStmt.setString(2, grade);
                            insertStmt.setInt(3, courseId);
                            insertStmt.setTimestamp(4, new Timestamp(System.currentTimeMillis()));
                            insertStmt.executeUpdate();
                            System.out.println(GREEN + "Grade added successfully for student " + studentId + " in course " + courseId + RESET);
                        }
                    }
                }
            }
        } catch (SQLException ex) {
            throw new GradeUpdateException("Error adding/updating grade: " + ex.getMessage(), ex);
        }
    }
    
    // Helper method to check if a student is enrolled in a course taught by the professor
    private boolean isCourseTaughtByProfessor(Connection conn, int studentId, int courseId, int professorId) throws SQLException {
        try (PreparedStatement stmt = conn.prepareStatement(SQLConstant.CHECK_PROFESSOR_STUDENT_ENROLLMENT)) {
            stmt.setInt(1, studentId);
            stmt.setInt(2, courseId);
            stmt.setInt(3, professorId);
            try (ResultSet rs = stmt.executeQuery()) {
                return rs.next(); // Returns true if the student is enrolled in the professor's course
            }
        }
    }
    public List<Course> getProfessorCourses(int professorId) {
        List<Course> courses = new ArrayList<>();
        try (Connection conn = DBUtils.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(SQLConstant.FETCH_PROFESSOR_COURSES)) {
            pstmt.setInt(1, professorId);
            pstmt.setInt(2, professorId);
            try (ResultSet rs = pstmt.executeQuery()) {
                while (rs.next()) {
                    Course course = new Course(rs.getInt("course_id"), rs.getString("course_code"),
    		                rs.getString("course_name"), rs.getString("professor_id"), rs.getString("professor_name"),rs.getInt("bill_amount"), rs.getInt("capacity"), rs.getInt("currentEnrollment"));
                    courses.add(course);
                }
            }
        } catch (SQLException ex) {
            // Log the error
        	System.out.println("SQLException generated:- "+ex.getMessage());
        }
        return courses;
    }
    
    public List<Student> getEnrolledStudentsInCourse(int courseId) {
        List<Student> students = new ArrayList<>();
        try (Connection conn = DBUtils.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(SQLConstant.FETCH_ENROLLED_STUDENTS)) {
            pstmt.setInt(1, courseId);

            try (ResultSet rs = pstmt.executeQuery()) {
                while (rs.next()) {
                    Student student = new Student(
                        rs.getString("first_name"),
                        rs.getString("last_name"),
                        rs.getString("gender"),
                        rs.getInt("age"),
                        rs.getDouble("tenth_percentage"),
                        rs.getDouble("twelfth_percentage"),
                        rs.getString("address"),
                        rs.getString("phone_number"),
                        rs.getString("email_id")
                    ); 
                    students.add(student);
                }
            }
        } catch (SQLException ex) {
            throw new EnrolledStudentsRetrievalException("Error fetching enrolled students for course: " + courseId, ex); // Throw custom exception
        }
        return students;
    }	/**
	 * Retrieves a list of student IDs enrolled in a specific course. This method
	 * includes SQL to query the database for enrolled students.
	 *
	 * @param courseId the ID of the course for which to view enrolled students
	 * @return a List of strings containing student IDs of enrolled students
	 */
    public List<Student> getEnrolledStudents(int professorId) {
        List<Student> students = new ArrayList<>();
        try (Connection conn = DBUtils.getConnection()) {
            try (PreparedStatement pstmt = conn.prepareStatement(SQLConstant.FETCH_ENROLLED_STUDENTS_PROFESSOR)) {
                pstmt.setInt(1, professorId);

                try (ResultSet rs = pstmt.executeQuery()) {
                    while (rs.next()) {
                        Student student = new Student(
                            rs.getString("first_name"),
                            rs.getString("last_name"),
                            rs.getString("gender"),
                            rs.getInt("age"),
                            rs.getDouble("tenth_percentage"),
                            rs.getDouble("twelfth_percentage"),
                            rs.getString("address"),
                            rs.getString("phone_number"),
                            rs.getString("email_id")
                        ); // Assuming you have a StudentDetails class
                        students.add(student);
                    }
                }
            }
        } catch (SQLException ex) {
            // Log the error: logger.error("Error fetching enrolled students: {}", e.getMessage());
        	System.out.println("SQLException generated"+ex.getMessage());
        }
        return students;
    }
    
    public int currentProfessorId() {
        String username = userDAO.getLoggedInUsername();

        // Check if username is null
        if(username==null || username.trim().isEmpty()){
            throw new RuntimeException("Error fetching professor ID: Logged-in username is null or empty");
        }

        try (Connection conn = DBUtils.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(SQLConstant.FETCH_PROFESSOR_ID)) {

            pstmt.setString(1, username);

            try (ResultSet rs = pstmt.executeQuery()) {
                if (rs.next()) {
                    int professorId = rs.getInt("user_id");// Debug: Print the fetched ID
                    return professorId;
                } else {
                    System.out.println("No professor found for username: " + username); // Debug: Indicate no matching professor
                }
            }
        } catch (SQLException ex) {
            System.out.println("Error fetching professor ID: " + ex.getMessage()); // More detailed error message
        }

        return -1; // Indicates professor not found
    }

}