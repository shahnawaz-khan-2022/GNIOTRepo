package com.gniot.crs.dao;

import java.sql.*;
import java.util.ArrayList;
import java.util.List;
import com.gniot.crs.bean.Course;
import com.gniot.crs.bean.Professor;
import com.gniot.crs.constant.SQLConstant;
import com.gniot.crs.exception.*;
import com.gniot.crs.utils.DBUtils;

/**
 * The AdminDAOImpl class implements the AdminDAOInterface. It provides the SQL
 * commands for each method to interact with the database for managing courses,
 * professors, and student approvals.
 */
public class AdminDAOImpl implements AdminDAOInterface {
	final String GREENBG = "\u001B[42m"; // ANSI escape code for green text
	final String REDBG = "\u001B[41m"; // ANSI escape code for red text
	final String RESET = "\u001B[0m";
	final String CYAN = "\u001B[96m";
	final String GREEN = "\u001B[32m"; // ANSI escape code for green text
	final String RED = "\u001B[31m"; // ANSI escape code for red text

	final String YELLOW = "\u001B[33m";
	public String errorMessage;

	/**
	 * Adds a new course to the course catalog. This method includes SQL to insert a
	 * new course record into the database.
	 *
	 * @param courseId   the ID of the course to be added
	 * @param courseName the name of the course to be added
	 * @param courseCode the code of the course to be added
	 */
	@Override
	public void addCourseToCatalog(int courseId, String courseName, String courseCode) {
	    try (Connection conn = DBUtils.getConnection();
	         PreparedStatement stmt = conn.prepareStatement(SQLConstant.INSERT_COURSES)) {
	        stmt.setInt(1, courseId);
	        stmt.setString(2, courseName);
	        stmt.setString(3, courseCode);
	        stmt.executeUpdate();
	    } catch (SQLException ex) {
	        // Log the error: logger.error("Error adding course to catalog: {}", e.getMessage());
	        throw new CourseAdditionException("Error adding course to catalog", ex); // Throw custom exception
	    }
	}

	/**
	 * Removes a course from the course catalog. This method includes SQL to delete
	 * the course record from the database.
	 *
	 * @param courseId the ID of the course to be removed
	 */

	@Override
	public void removeCourseFromCatalog(int courseId) {
	    try (Connection conn = DBUtils.getConnection();
	         PreparedStatement stmt = conn.prepareStatement(SQLConstant.REMOVE_COURSES)) {
	        stmt.setInt(1, courseId);
	        int rowsDeleted = stmt.executeUpdate();
	        if (rowsDeleted == 0) {
	            // Specific exception for course not found
	            throw new CourseRemovalException("Course with ID " + courseId + " not found in the catalog."); 
	        } else {
	            System.out.println(GREEN + "Course with ID " + courseId + " has been removed from the catalog." + RESET);
	        }
	    } catch (SQLException ex) {
	        // Log the error: logger.error("Error removing course from catalog: {}", e.getMessage());
	        throw new CourseRemovalException("Error removing course from catalog", ex);
	    }
	}

	public boolean setBillAmount(int courseId, int newBillAmount) {
	    try (Connection conn = DBUtils.getConnection()) {
	        // 1. Check if course exists
	        if (!courseExists(conn, courseId)) {
	            throw new CourseNotFoundException("Error: Course not found with ID " + courseId); // Use CourseNotFoundException
	        }

	        // 2. Update the bill amount
	        try (PreparedStatement pstmt = conn.prepareStatement(SQLConstant.UPDATE_BILL)) {
	            pstmt.setInt(1, newBillAmount);
	            pstmt.setInt(2, courseId);
	            int rowsUpdated = pstmt.executeUpdate();
	            return rowsUpdated > 0; 
	        }
	    } catch (SQLException ex) {
	        // Log the error: logger.error("Error setting bill amount: {}", e.getMessage());
	        throw new BillAmountUpdateException("Error setting bill amount for course ID: " + courseId, ex); // Throw custom exception
	    }
	}

	/**
	 * Assigns a course to a professor. This method includes SQL to update the
	 * course record with the professor's information in the database.
	 *
	 * @param courseId  the ID of the course to be assigned
	 * @param professor the name of the professor to whom the course is assigned
	 */
	public boolean assignCourseToProfessor(int professorId, int courseId) {
	    try (Connection conn = DBUtils.getConnection()) {
	        // 1. Check if professor exists
	        if (!professorExists(conn, professorId)) {
	            throw new ProfessorNotFoundException(RED + "Error: Professor not found with ID " + professorId + RESET); // Throw custom exception
	        }

	        // 2. Check if course exists
	        if (!courseExists(conn, courseId)) {
	            throw new CourseNotFoundException(RED + "Error: Course not found with ID " + courseId + RESET); // Throw custom exception
	        }

	        // 3. Update the course table to assign the professor
	        try (PreparedStatement pstmt = conn.prepareStatement(SQLConstant.ASSIGN_COURSE)) {
	            pstmt.setInt(1, professorId);
	            pstmt.setInt(2, professorId);   // Get the professor's name
	            pstmt.setInt(3, courseId);
	            int rowsUpdated = pstmt.executeUpdate();
	            return rowsUpdated > 0; // Return true if the assignment was successful
	        }
	    } catch (SQLException ex) {
	        // Log the error: logger.error("Error assigning course to professor: {}", e.getMessage());
	        throw new CourseAssignmentException("Error assigning course to professor", ex);
	    }
	}
	public boolean removeProfessorFromCourse( int courseId) {
	    try (Connection conn = DBUtils.getConnection()) {
	        conn.setAutoCommit(false);

	        try (PreparedStatement pstmt = conn.prepareStatement(SQLConstant.REMOVE_PROFESSOR_FROM_COURSE)) {// Assuming professor_id column exists in course table
	            pstmt.setInt(1, courseId);
	            int rowsUpdated = pstmt.executeUpdate();

	            if (rowsUpdated > 0) {
	                conn.commit();
	                System.out.println(GREEN + "Professor removed from course successfully." + RESET);
	                return true;
	            } else {
	                throw new RemoveProfessorFromCourseException("Professor not found in this course or was not the assigned professor.");
	            }

	        } catch (SQLException ex) {
	            // Log the error: logger.error("Error removing professor from course: {}", e.getMessage());
	            conn.rollback(); // Rollback the transaction on error
	            throw new RemoveProfessorFromCourseException("Error removing professor from course", ex);
	        } finally {
	            conn.setAutoCommit(true); // Reset autocommit
	        }
	    } catch (SQLException ex) {
	        // Log the error: logger.error("Error removing professor from course (connection issue): {}", e.getMessage());
	        throw new RemoveProfessorFromCourseException("Error removing professor from course: Database connection issue.", ex);
	    }
	}

	public List<Professor> getAllProfessors() {
		List<Professor> professors = new ArrayList<>();
		try (Connection conn = DBUtils.getConnection();
				PreparedStatement pstmt = conn.prepareStatement(SQLConstant.FETCH_PROFESSOR); // Fetch all professors
				ResultSet rs = pstmt.executeQuery()) {

			while (rs.next()) {
				Professor professor = new Professor(rs.getInt("professor_id"),
						rs.getString("first_name"), rs.getString("last_name"), rs.getString("gender"), rs.getInt("age"),
						rs.getString("address"), rs.getString("phone_number"), rs.getString("email_id")); // Assuming
																											// you have
																											// a
																											// Professor
																											// class
				professors.add(professor);
			}
		} catch (SQLException ex) {
			// Log the error: logger.error("Error fetching professors: {}", e.getMessage());
			throw new RuntimeException("Error fetching professors", ex);
		}
		return professors;
	}

	private boolean professorExists(Connection conn, int professorId) throws SQLException {
		try (PreparedStatement stmt = conn.prepareStatement(SQLConstant.PROFESSOR_EXIST)) {
			stmt.setInt(1, professorId);
			try (ResultSet rs = stmt.executeQuery()) {
				return rs.next(); // Returns true if a professor with this ID exists
			}
		}
	}

	// Helper method to check if a course exists
	private boolean courseExists(Connection conn, int courseId) throws SQLException {
		try (PreparedStatement stmt = conn.prepareStatement(SQLConstant.COURSE_EXIST)) {
			stmt.setInt(1, courseId);
			try (ResultSet rs = stmt.executeQuery()) {
				return rs.next(); // Returns true if a course with this ID exists
			}
		}
	}

	/**
	 * Adds a new professor to the system. This method includes SQL to insert a new
	 * professor record into the database.
	 *
	 * @param professor the Professor object containing the details of the professor
	 *                  to be added
	 */

    public boolean approveProfessor(int professorId) {
        try (Connection conn = DBUtils.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(SQLConstant.APPROVE_PROFESSOR)) {

            pstmt.setInt(1, professorId);
            int rowsUpdated = pstmt.executeUpdate();

            // Check if any rows were updated
            if (rowsUpdated == 0) {
                throw new ProfessorNotFoundException("Error approving professor: Professor with ID " + professorId + " not found.");
            }

            return true; 
        } catch (SQLException ex) {
            // Log the error: logger.error("Error approving professor: {}", e.getMessage());
            throw new RuntimeException("Error approving professor", ex);
        }
    }

    public boolean removeProfessor(int professorId) throws ProfessorRemovalException  {
        try (Connection conn = DBUtils.getConnection()) {
            conn.setAutoCommit(false); // Start a transaction

            try {
                // 1. Reset professor column in the course table for any assigned courses
                try (PreparedStatement pstmt = conn.prepareStatement(SQLConstant.REMOVE_PROFESSOR_FROM_COURSE)) {
                    pstmt.setInt(1, professorId);
                    pstmt.executeUpdate();
                }

                // 2. Delete from professors table
                try (PreparedStatement pstmt = conn.prepareStatement(SQLConstant.REMOVE_PROFESSOR_FROM_PROFESSOR)) {
                    pstmt.setInt(1, professorId);
                    int rowsDeletedFromProfessors = pstmt.executeUpdate();

                    if (rowsDeletedFromProfessors == 0) {
                        throw new ProfessorNotFoundException("Professor not found with ID: " + professorId);
                    }
                }

                // 3. Delete from users table
                try (PreparedStatement pstmt = conn.prepareStatement(SQLConstant.REMOVE_PROFESSOR_FROM_USER)) {
                    pstmt.setInt(1, professorId);
                    int rowsDeletedFromUsers = pstmt.executeUpdate();

                    if (rowsDeletedFromUsers == 0) {
                        throw new UserRemovalException("Error deleting user associated with professor ID: " + professorId); 
                    }
                }

                conn.commit();
                return true;
            } catch (SQLException | ProfessorNotFoundException | UserRemovalException ex) {
                conn.rollback();
                // Log the error: logger.error("Error removing professor: {}", e);  
                throw new ProfessorRemovalException("Error removing professor.", ex);
            } finally {
                conn.setAutoCommit(true);
            }
        } catch (SQLException ex) {
            // Log the error: logger.error("Error removing professor (connection issue): {}", e);  
            throw new ProfessorRemovalException("Error removing professor: Database connection error.", ex); // Top-level exception
        }
    }

	/**
	 * Approves a student's registration. This method includes SQL to update the
	 * student's status in the database.
	 *
	 * @param Username the username of the student to be approved
	 */

public void approveStudent(String username) {
    try (Connection conn = DBUtils.getConnection();
         PreparedStatement stmt = conn.prepareStatement(SQLConstant.APPROVE_STUDENT)) {

        stmt.setString(1, username);
        int rowsUpdated = stmt.executeUpdate();

        if (rowsUpdated == 0) { // Check if the student exists
            throw new StudentNotFoundException("Student with username '" + username + "' not found.");
        }

    } catch (SQLException ex) {
        // Log the error: logger.error("Error approving student: {}", e.getMessage());
        throw new StudentApprovalException("Error approving student with username '" + username + "'", ex);
    }
}

	/**
	 * Retrieves a list of usernames of students whose registration is pending
	 * approval. This method includes SQL to query the database for pending
	 * students.
	 *
	 * @return a List of strings containing usernames of pending students
	 */
@Override
public List<String> getPendingStudents() {
    List<String> pendingStudents = new ArrayList<>();

    try (Connection conn = DBUtils.getConnection();
         PreparedStatement stmt = conn.prepareStatement(SQLConstant.GET_PENDING_STUDENTS);
         ResultSet rs = stmt.executeQuery()) {

        while (rs.next()) {
            String studentInfo = String.format("| %-7s | %-10s |", rs.getString("user_id"), rs.getString("username"));
            pendingStudents.add(studentInfo);
        }

    } catch (SQLException ex) {
        // Log the error: logger.error("Error fetching pending students: {}", e.getMessage());
        throw new PendingStudentsRetrievalException("Error fetching pending students", ex); // Throw custom exception
    }
    return pendingStudents;
}

public List<String> getPendingProfessor() {
    List<String> pendingProfessor = new ArrayList<>();
    try (Connection conn = DBUtils.getConnection();
         PreparedStatement stmt = conn.prepareStatement(SQLConstant.GET_PENDING_PROFESSOR);
         ResultSet rs = stmt.executeQuery()) {

    	printHorizontalLine(20);
		System.out.println("Pending Professors:");
        printHorizontalLine(25);
        System.out.printf("| %-7s | %-10s |\n", "User ID", "User Name");
        printHorizontalLine(25);

        while (rs.next()) {
            String professorInfo = String.format("| %-7s | %-10s |", rs.getString("user_id"), rs.getString("username"));
            pendingProfessor.add(professorInfo);
        }

    } catch (SQLException ex) {
        // Log the error: logger.error("Error fetching pending professors: {}", e.getMessage());
        throw new PendingProfessorsRetrievalException("Error fetching pending professors", ex); // Throw custom exception
    }
    return pendingProfessor;
}

	/**
	 * Retrieves the course catalog. This method includes SQL to query the database
	 * for the list of all courses.
	 *
	 * @return a List of Course objects representing the course catalog
	 */
public List<Course> getCourseCatalog() {
    List<Course> courses = new ArrayList<>();

    try (Connection conn = DBUtils.getConnection();
         Statement stmt = conn.createStatement();
         ResultSet rs = stmt.executeQuery(SQLConstant.GET_COURSE_CATALOG)) {

        while (rs.next()) {
            courses.add(new Course(
                rs.getInt("course_id"),
                rs.getString("course_code"),
                rs.getString("course_name"),
                rs.getString("professor_id"),
                rs.getString("professor_name"),
                rs.getInt("bill_amount"),
                rs.getInt("capacity"),
                rs.getInt("currentEnrollment")
            ));
        }

    } catch (SQLException ex) {
        // Log the error: logger.error("Error retrieving course catalog: {}", e.getMessage());
        throw new CourseCatalogRetrievalException("Error retrieving course catalog", ex); 
    }

    return courses;
}

	private void printHorizontalLine(int... widths) {
		StringBuilder line = new StringBuilder("+");
		for (int width : widths) {
			for (int i = 0; i < width; i++) {
				line.append("-");
			}
			line.append("-+");
		}
		System.out.println(line);
	}

}
