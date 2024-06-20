/**
 * 
 */
package com.gniot.crs.dao;

import java.util.List;

import com.gniot.crs.bean.Course;
import com.gniot.crs.bean.Grade;
import com.gniot.crs.bean.Payment;
import com.gniot.crs.bean.Student;

/**
 * The StudentDAOInterface defines the data access methods required for student
 * operations in the course registration system. These methods will include SQL
 * commands for interacting with the database to manage course browsing,
 * password changes, account information, and course enrollment.
 */
public interface StudentDAOInterface {
	/**
	 * Retrieves the list of all available courses from the course catalog. This
	 * method will include SQL to query the database for the course catalog.
	 *
	 * @return a List of Course objects representing the available courses
	 */
	List<Course> browseCatalogForCourses();

	/**
	 * Retrieves account information for a student. This method will include SQL to
	 * query the database for the student's details.
	 *
	 * @param studentId the ID of the student whose account information is to be
	 *                  retrieved
	 * @return a StudentDetails object containing the student's account information
	 */

	Student accountInfo(int studentId);
	public int currentStudentId();

	/**
	 * Adds a course to a student's enrolled courses. This method will include SQL
	 * to insert the course enrollment record in the database.
	 *
	 * @param studentId the ID of the student enrolling in the course
	 * @param courseId  the ID of the course to be added
	 */
	public List<Grade> viewGrades(int studentId);
	public boolean removeCourse(int studentId, int courseId);
	public Course getCourseById(int courseId);
	public void updateCourseEnrollment(int courseId, int newEnrollment);
	   public void updateDueAmountsForStudent(int studentId);
	 public void updateDueAmount(int paymentId, double newDueAmount);
	void addCourse(int studentId, int courseId);
	public void recordPayment(int studentId, double amount, String paymentMethod, String status, double dueAmount);
	public double calculateTotalBillAmount(int studentId);
	public int getLatestPaymentId(int studentId);
	public List<Payment> getPaymentHistory(int studentId);
	public int getLastInsertedPaymentId();
	 public double getTotalPaidAmount(int studentId);
	 public List<Course> getEnrolledCourses(int studentId);
	 public boolean isStudentEnrolled(int studentId, int courseId);
}
