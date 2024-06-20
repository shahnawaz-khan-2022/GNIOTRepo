package com.gniot.crs.dao;

import java.sql.Connection;
import java.sql.Date;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.sql.Timestamp;
import java.util.ArrayList;
import java.util.List;

import com.gniot.crs.bean.*;
import com.gniot.crs.constant.SQLConstant;
import com.gniot.crs.exception.*;
import com.gniot.crs.utils.DBUtils;

/**
 * Implementation of StudentDAOInterface. This class provides the SQL
 * implementation for student operations such as browsing courses, changing
 * passwords, retrieving account info, and managing course enrollments.
 */
public class StudentDAOImpl implements StudentDAOInterface {
	final String GREEN = "\u001B[32m"; // ANSI escape code for green text
	final String RED = "\u001B[31m"; // ANSI escape code for red text

	final String YELLOW = "\u001B[33m";
	final String RESET = "\u001B[0m";

	public String errorMessage;
	/**
	 * Retrieves the list of all available courses from the course catalog. This
	 * method includes SQL to query the database for the course catalog.
	 *
	 * @return a List of Course objects representing the available courses
	 */

	UserDAOImpl daoImpl = new UserDAOImpl(); // create an instance of UserDAOImpl // log the user in
	String username = daoImpl.getLoggedInUsername();

	@Override
	public List<Course> browseCatalogForCourses() {
		List<Course> courses = new ArrayList<>();
		try (Connection conn = DBUtils.getConnection();
				PreparedStatement pstmt = conn.prepareStatement(SQLConstant.DISPLAY_COURSES)) {

			try (ResultSet rs = pstmt.executeQuery()) {
				while (rs.next()) {
					courses.add(new Course(rs.getInt("course_id"), rs.getString("course_code"),
							rs.getString("course_name"), rs.getString("professor_id"), rs.getString("professor_name"),
							rs.getInt("bill_amount"), rs.getInt("capacity"), rs.getInt("currentEnrollment")));
				}
			}
		} catch (SQLException ex) {
			// Throw your custom exception
			throw new CourseCatalogException("Error fetching course catalog", ex);
		}
		return courses;
	}

	/**
	 * Adds a course to a student's enrolled courses. This method includes SQL to
	 * insert the course enrollment record in the database.
	 *
	 * @param studentId the ID of the student enrolling in the course
	 * @param courseId  the ID of the course to be added
	 */

	public void addCourse(int studentId, int courseId) {
		try (Connection conn = DBUtils.getConnection()) {
			// Get connection first, then manage transactions

			// 1. Check if student exists
			if (!studentExists(conn, studentId)) {
				throw new StudentNotFoundException("Error: Student not found with id: " + studentId);
			}

			// 2. Check if course exists
			if (!courseExists(conn, courseId)) {
				throw new CourseNotFoundException("Error: Course not found with id: " + courseId);
			}

			// 3. Check if student is already enrolled in this course
			if (isStudentEnrolled(studentId, courseId)) {
				throw new EnrollmentException("Student is already enrolled in this course.", null);
			}

			// 4. Check if student can enroll in more courses
			int enrolledCoursesCount = getEnrolledCoursesCount(studentId);
			if (enrolledCoursesCount >= 4) {
				throw new EnrollmentException("Student cannot enroll in more than 4 courses.", null);
			}

			conn.setAutoCommit(false); // Start a transaction
			try {
				// 5. Insert the enrollment record
				try (PreparedStatement insertStmt = conn.prepareStatement(SQLConstant.INSERT_ENROLLED_COURSES)) {
					insertStmt.setInt(1, studentId);
					insertStmt.setInt(2, courseId);
					insertStmt.setTimestamp(3, new Timestamp(System.currentTimeMillis()));
					insertStmt.executeUpdate();
				}

				// 6. Get the updated course object
				Course course = getCourseById(courseId);
				if (course == null) {
					throw new SQLException("Course not found after adding enrollment."); // More specific error
				}

				// 7. Update the course enrollment count (with null check)
				int newEnrollment = course.getCurrentEnrollment() + 1; // Default 0 if null
				updateCourseEnrollment(courseId, newEnrollment);

				conn.commit(); // Commit only if everything is successful
				System.out.println(GREEN + "Course assigned successfully to student " + studentId + RESET);
			} catch (SQLException ex) {
				conn.rollback();
				throw new EnrollmentException("Database error during enrollment", ex); // Include cause
			} finally {
				conn.setAutoCommit(true);
			}
		} catch (SQLException ex) {
			// Log the error
			throw new EnrollmentException("Could not connect to the database.", ex);
		}
	}

	public Course getCourseById(int courseId) {
		try (Connection conn = DBUtils.getConnection();
				PreparedStatement pstmt = conn.prepareStatement(SQLConstant.GET_COURSE_BY_ID)) {

			pstmt.setInt(1, courseId);
			try (ResultSet rs = pstmt.executeQuery()) {
				if (rs.next()) {
					return new Course(rs.getInt("course_id"), rs.getString("course_code"), rs.getString("course_name"),
							rs.getString("professor_id"), rs.getString("professor_name"), rs.getInt("bill_amount"),
							rs.getInt("capacity"), rs.getInt("currentEnrollment"));
				}
			}
		} catch (SQLException ex) {
			throw new RuntimeException("Error fetching course by ID", ex);
		}
		return null;
	}

	// Method to update course enrollment in the database
	public void updateCourseEnrollment(int courseId, int newEnrollment) {
		try (Connection conn = DBUtils.getConnection();
				PreparedStatement pstmt = conn.prepareStatement(SQLConstant.UPDATE_COURSE_ENROLLMENT)) {
			pstmt.setInt(1, newEnrollment);
			pstmt.setInt(2, courseId);
			pstmt.executeUpdate();
		} catch (SQLException ex) {
			throw new RuntimeException("Error updating course enrollment", ex);
		}
	}

	/**
	 * Removes a course from a student's enrolled courses. This method includes SQL
	 * to delete the course enrollment record from the database.
	 *
	 * @param studentId the ID of the student whose course is to be removed
	 * @param courseId  the ID of the course to be removed
	 */
	// Remove Course
	public boolean removeCourse(int studentId, int courseId) {
		try (Connection conn = DBUtils.getConnection()) {
			conn.setAutoCommit(false); // Start a transaction
			try {
				// Delete the enrollment record
				try (PreparedStatement deleteStmt = conn.prepareStatement(SQLConstant.DELETE_ENROLLED_COURSES)) {
					deleteStmt.setInt(1, studentId);
					deleteStmt.setInt(2, courseId);
					int rowsDeleted = deleteStmt.executeUpdate();
					if (rowsDeleted == 0) {
						throw new EnrollmentException("Course not found or you are not enrolled in it.", null); // Specific
																												// exception
																												// for
																												// enrollment
																												// issue
					}
				}

				// Get the updated course object
				Course course = getCourseById(courseId);
				if (course == null) {
					throw new CourseNotFoundException("Course not found after removing enrollment."); // Specific
																										// exception for
																										// course not
																										// found
				}

				// Decrement enrollment and update the database
				updateCourseEnrollment(courseId, course.getCurrentEnrollment() - 1);

				conn.commit();
				System.out.println(GREEN + "Course removed successfully." + RESET);
				return true;
			} catch (SQLException | EnrollmentException | CourseNotFoundException ex) { // Catch multiple exception
																						// types
				conn.rollback();
				System.out.println(RED + "Error during course removal: " + ex.getMessage() + RESET);
				return false;
			} finally {
				conn.setAutoCommit(true); // Restore auto-commit mode
			}
		} catch (SQLException ex) {
			System.out
					.println(RED + "Error: Database error occurred during course removal. " + ex.getMessage() + RESET);
			return false;
		}
	}

	/**
	 * Retrieves the grades of a student. This method includes SQL to query the
	 * database for the student's grades.
	 *
	 * @param studentId the ID of the student whose grades are to be retrieved
	 * @return a List of Grade objects representing the student's grades
	 */
	public List<Grade> viewGrades(int studentId) {
		List<Grade> grades = new ArrayList<>();
		try (Connection conn = DBUtils.getConnection();
				PreparedStatement pstmt = conn.prepareStatement(SQLConstant.VIEW_GRADE)) {

			pstmt.setInt(1, studentId);
			try (ResultSet rs = pstmt.executeQuery()) {
				if (!rs.next()) { // Check if there are no grades found
					throw new GradeRetrievalException(YELLOW + "No grades found for this student." + RESET);
				}
				do { // Loop through all grades using do-while loop
					Grade grade = new Grade(rs.getInt("grade_id"), rs.getInt("student_id"), rs.getString("grades"),
							rs.getInt("course_id"), rs.getString("course_code"), rs.getString("course_name"),
							rs.getTimestamp("created_at"));
					grades.add(grade);
				} while (rs.next());
			}
		} catch (SQLException ex) {
			// Log the error for debugging
			throw new GradeRetrievalException("Error fetching grades: " + ex.getMessage(), ex);
		}
		return grades;
	}

	/**
	 * Retrieves account information for a student. This method includes SQL to
	 * query the database for the student's details.
	 *
	 * @param studentId the ID of the student whose account information is to be
	 *                  retrieved
	 * @return a StudentDetails object containing the student's account information
	 */
	@Override
	public Student accountInfo(int studentId) {
		try (Connection conn = DBUtils.getConnection();
				PreparedStatement pstmt = conn.prepareStatement(SQLConstant.FETCH_STUDENT_DETAILS)) {

			pstmt.setInt(1, studentId);

			try (ResultSet rs = pstmt.executeQuery()) {
				if (rs.next()) {
					return new Student(rs.getString("first_name"), rs.getString("last_name"),
							rs.getString("gender"), rs.getInt("age"), rs.getDouble("tenth_percentage"),
							rs.getDouble("twelfth_percentage"), rs.getString("address"), rs.getString("phone_number"),
							rs.getString("email_id"));
				} else {
					throw new StudentNotFoundException("Student not found with id: " + studentId);
				}
			}
		} catch (SQLException ex) {
			// Log the error: logger.error("Error fetching account info: {}",
			// e.getMessage());
			throw new RuntimeException("Error fetching account info", ex); // More generic exception for other errors
		}
	}

	public double calculateTotalBillAmount(int studentId) {
		try (Connection conn = DBUtils.getConnection();
				PreparedStatement pstmt = conn.prepareStatement(SQLConstant.CALCULATE_BILL)) {

			pstmt.setInt(1, studentId);

			try (ResultSet rs = pstmt.executeQuery()) {
				if (rs.next()) {
					return rs.getDouble("total_amount");
				} else {
					// No courses enrolled, return 0
					return 0.0;
				}
			}
		} catch (SQLException ex) {
			// Log the error: logger.error("Error calculating total bill amount for
			// studentId {}: {}", studentId, e.getMessage());
			throw new BillCalculationException("Error calculating total bill amount for student: " + studentId, ex); // Throw
																														// custom
																														// exception
		}
	}

	public void recordPayment(int studentId, double amount, String paymentMethod, String status, double dueAmount) {
		try (Connection conn = DBUtils.getConnection();
				PreparedStatement pstmt = conn.prepareStatement(SQLConstant.RECORD_PAYMENT, // Using the constant from
																							// SQLConstant
						Statement.RETURN_GENERATED_KEYS)) {

			pstmt.setInt(1, studentId);
			pstmt.setDouble(2, amount);
			pstmt.setDate(3, new Date(System.currentTimeMillis()));
			pstmt.setString(4, paymentMethod);
			pstmt.setString(5, status);
			pstmt.setDouble(6, dueAmount);
			pstmt.executeUpdate();

			try (ResultSet generatedKeys = pstmt.getGeneratedKeys()) {
				if (generatedKeys.next()) {
					lastInsertedPaymentId = generatedKeys.getInt(1);
				} else {
					throw new PaymentHistoryException("Creating payment failed, no ID obtained."); // Custom exception
				}
			}

			System.out.println(
					GREEN + "Payment of Rs." + amount + " recorded successfully for student " + studentId + RESET);

		} catch (SQLException ex) {
			// Log the error
			System.err.println(RED + "Error recording payment: " + ex.getMessage() + RESET);
			throw new PaymentHistoryException("Error recording payment", ex); // Wrap SQLException
		}
	}

	public void recordPaymentDetails(int studentId, String paymentMethod, PaymentDetails details) {
		try (Connection conn = DBUtils.getConnection()) {
			if (paymentMethod.equals("Credit Card") || paymentMethod.equals("Debit Card")) {
				// Insert or update card details for credit/debit card
				try (PreparedStatement pstmt = conn.prepareStatement(SQLConstant.INSERT_CARD)) {
					pstmt.setString(1, details.getCardNumber());
					pstmt.setString(2, details.getExpiryDate());
					pstmt.setString(3, details.getCvv());
					pstmt.setInt(4, studentId);
					pstmt.executeUpdate();
				}
			} else if (paymentMethod.equals("Net Banking")) {
				// Insert or update bank details for net banking
				try (PreparedStatement pstmt = conn.prepareStatement(SQLConstant.INSERT_NETBANK)) {
					pstmt.setString(1, details.getBankName());
					pstmt.setInt(2, studentId);
					pstmt.executeUpdate();
				}
			}
		} catch (SQLException e) {
			// Log the error
			throw new RuntimeException("Error recording payment details", e);
		}
	}

	private int lastInsertedPaymentId;

	public int getLastInsertedPaymentId() {
		return lastInsertedPaymentId;
	}

	public int getLatestPaymentId(int studentId) throws LatestPaymentNotFoundException {
		try (Connection conn = DBUtils.getConnection();
				PreparedStatement pstmt = conn.prepareStatement(SQLConstant.FETCH_PAYMENTID)) {

			pstmt.setInt(1, studentId);

			try (ResultSet rs = pstmt.executeQuery()) {
				if (rs.next()) {
					return rs.getInt("payment_id");
				} else {
					throw new LatestPaymentNotFoundException("No payment found for student with ID: " + studentId);
				}
			}
		} catch (SQLException ex) {
			// Log the error
			System.out.println("Error fetching latest payment id: " + ex.getMessage());
			throw new LatestPaymentNotFoundException("Error fetching latest payment", ex);
		}
	}

	// Updated method to update due_amount
	public void updateDueAmount(int paymentId, double newDueAmount) {
		try (Connection conn = DBUtils.getConnection();
				PreparedStatement pstmt = conn.prepareStatement(SQLConstant.UPDATE_DUES)) {

			pstmt.setDouble(1, newDueAmount);
			pstmt.setInt(2, paymentId);
			pstmt.executeUpdate();
		} catch (SQLException ex) {
			// Log the error: logger.error("Error updating due amount for paymentId {}: {}",
			// paymentId, e.getMessage());
			throw new DueAmountUpdateException("Error updating due amount for payment ID: " + paymentId, ex);
		}
	}

	public void updateDueAmountsForStudent(int studentId) {
		try (Connection conn = DBUtils.getConnection()) {

			try (PreparedStatement pstmt = conn.prepareStatement(SQLConstant.UPDATE_DUE_AMOUNTS_LATEST_PAYMENT)) {
				pstmt.setInt(1, studentId);
				pstmt.executeQuery();
			}

			try (PreparedStatement pstmt = conn.prepareStatement(SQLConstant.UPDATE_DUE_AMOUNTS_PREVIOUS_PAYMENTS)) {
				pstmt.setInt(1, studentId);
				pstmt.executeUpdate();
			}
		} catch (SQLException ex) {
			// Log the error: logger.error("Error updating due amounts for studentId {}:
			// {}", studentId, e.getMessage());
			throw new StudentDueUpdateException("Error updating due amounts for student: " + studentId, ex);
		}
	}

	public List<Payment> getPaymentHistory(int studentId) {
		List<Payment> payments = new ArrayList<>();
		try (Connection conn = DBUtils.getConnection();
				PreparedStatement pstmt = conn.prepareStatement(SQLConstant.PAYMENT_HISTORY)) {

			pstmt.setInt(1, studentId);

			try (ResultSet rs = pstmt.executeQuery()) {
				while (rs.next()) {
					Payment payment = new Payment(rs.getInt("payment_id"), rs.getInt("student_id"),
							rs.getDouble("amount"), rs.getDate("payment_date"), rs.getString("payment_method"),
							rs.getString("status"), rs.getDouble("total_amount"), rs.getDouble("initial_due"));
					payments.add(payment);
				}
			} catch (SQLException ex) {
				// Log the error: logger.error("Error fetching payment history for studentId {}:
				// {}", studentId, ex.getMessage());
				throw new PaymentHistoryRetrievalException("Error fetching payment history for student: " + studentId,
						ex);
			}
		} catch (SQLException ex) {
			// Log the error: logger.error("Error connecting to database to fetch payment
			// history: {}", e.getMessage());
			throw new PaymentHistoryRetrievalException("Error connecting to database to fetch payment history", ex);
		}
		return payments;
	}

	public double getTotalPaidAmount(int studentId) {
		try (Connection conn = DBUtils.getConnection();
				PreparedStatement pstmt = conn.prepareStatement(SQLConstant.TOTAL_PAID_AMOUNT)) {

			pstmt.setInt(1, studentId);

			try (ResultSet rs = pstmt.executeQuery()) {
				if (rs.next()) {
					return rs.getDouble("total_paid");
				} else {
					// No payments found for the student
					return 0.0; // Or you can throw a custom exception here if needed
				}
			}
		} catch (SQLException ex) {
			// Log the error
			System.err.println(RED + "Error calculating total paid amount: " + ex.getMessage() + RESET); // More
																											// specific
																											// error
																											// message
			throw new TotalPaidAmountRetrievalException("Error fetching total paid amount for student: " + studentId,
					ex);
		}
	}

	/**
	 * Retrieves the student ID based on the username. This method includes SQL to
	 * query the database for the student ID.
	 *
	 * @param username the username of the student whose ID is to be retrieved
	 * @return the student ID or -1 if not found
	 */
	public int currentStudentId() throws StudentIdRetrievalException { // Declare that the method throws the exception
		String username = daoImpl.getLoggedInUsername();
		if (username == null) {
			throw new StudentIdRetrievalException("Error fetching student ID: Logged-in username is null"); // Specific
																											// exception
		}

		try (Connection conn = DBUtils.getConnection();
				PreparedStatement pstmt = conn.prepareStatement(SQLConstant.FETCH_STUDENTID)) {
			pstmt.setString(1, username);

			try (ResultSet rs = pstmt.executeQuery()) {
				if (rs.next()) {
					return rs.getInt("user_id");
				} else {
					throw new StudentIdRetrievalException(
							"Error fetching student ID: No student found for username " + username); // Specific
																										// exception
				}
			}
		} catch (SQLException ex) {
			// Log the error: logger.error("Error fetching student ID: {}", e.getMessage());
			throw new StudentIdRetrievalException("Error fetching student ID", ex); // Wrap SQLException
		}
	}

	public List<Course> getEnrolledCourses(int studentId) {
		List<Course> enrolledCourses = new ArrayList<>();
		try (Connection conn = DBUtils.getConnection();
				PreparedStatement pstmt = conn.prepareStatement(SQLConstant.FETCH_STUDENT_ENROLLLED_COURSES)) {
			pstmt.setInt(1, studentId);

			try (ResultSet rs = pstmt.executeQuery()) {
				while (rs.next()) {
					Course course = new Course(rs.getInt("course_id"), rs.getString("course_code"),
							rs.getString("course_name"), rs.getString("professor_id"), rs.getString("professor_name"),
							rs.getInt("bill_amount"), rs.getInt("capacity"), rs.getInt("currentEnrollment"));
					enrolledCourses.add(course);
				}
			}
		} catch (SQLException ex) {
			// Log the error: logger.error("Error fetching enrolled courses for studentId
			// {}: {}", studentId, e.getMessage());
			throw new EnrolledCoursesRetrievalException("Error fetching enrolled courses for student: " + studentId,
					ex);
		}
		return enrolledCourses;
	}

	public boolean isStudentEnrolled(int studentId, int courseId) {
		try (Connection conn = DBUtils.getConnection(); // Establish connection here
				PreparedStatement stmt = conn.prepareStatement(SQLConstant.CHECK_STUDENT_ENROLLLED)) {
			stmt.setInt(1, studentId);
			stmt.setInt(2, courseId);
			try (ResultSet rs = stmt.executeQuery()) {
				return rs.next(); // Returns true if the student is enrolled
			}
		} catch (SQLException ex) {
			// Log the error
			throw new RuntimeException("Error checking enrollment", ex); // Proper exception handling
		}
	}

	private int getEnrolledCoursesCount(int studentId) { // Removed Connection conn as argument
		try (Connection conn = DBUtils.getConnection(); // Establish connection within the method
				PreparedStatement stmt = conn.prepareStatement(SQLConstant.COUNT_STUDENT_ENROLLLED)) {
			stmt.setInt(1, studentId);
			try (ResultSet rs = stmt.executeQuery()) {
				if (rs.next()) {
					return rs.getInt("course_count");
				} else {
					return 0; // No enrollments found for this student
				}
			}
		} catch (SQLException ex) {
			// Log the error: logger.error("Error getting enrolled course count for
			// studentId {}: {}", studentId, e.getMessage());
			throw new EnrolledCoursesCountException("Error getting enrolled course count for student: " + studentId,
					ex);
		}
	}

	private boolean studentExists(Connection conn, int studentId) throws SQLException {
		// Check for student existence in 'students' table
		try (PreparedStatement stmt = conn.prepareStatement(SQLConstant.STUDENT_EXISTS)) {
			stmt.setInt(1, studentId);
			try (ResultSet rs = stmt.executeQuery()) {
				return rs.next(); // Returns true if a student with this ID exists
			}
		}
	}

	private boolean courseExists(Connection conn, int courseId) throws SQLException {
		// Check for course existence in 'course' table
		try (PreparedStatement stmt = conn.prepareStatement(SQLConstant.COURSE_EXIST)) {
			stmt.setInt(1, courseId);
			try (ResultSet rs = stmt.executeQuery()) {
				return rs.next(); // Returns true if a course with this ID exists
			}
		}
	}

}