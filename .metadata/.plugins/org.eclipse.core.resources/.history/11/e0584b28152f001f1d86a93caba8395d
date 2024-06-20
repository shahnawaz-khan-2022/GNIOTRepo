package com.gniot.crs.dao;

import java.util.List;

import com.gniot.crs.bean.Course;
import com.gniot.crs.bean.Professor;
import com.gniot.crs.exception.ProfessorRemovalException;
/**
 * The AdminDAOInterface defines the data access methods
 * that are required for admin operations in the course
 * registration system. It includes methods to manage courses,
 * professors, and student approvals.
 */
public interface AdminDAOInterface {
	/**
     * Adds a new course to the course catalog.
     *s
     * @param courseId the ID of the course to be added
     * @param courseName the name of the course to be added
     * @param courseCode the code of the course to be added
     */
    void addCourseToCatalog(int courseId, String courseName, String courseCode);
    /**
     * Removes a course from the course catalog.
     *
     * @param courseId the ID of the course to be removed
     */
    void removeCourseFromCatalog(int courseId);
    /**
     * Assigns a course to a professor.
     *
     * @param courseId the ID of the course to be assigned
     * @param professor the name of the professor to whom the course is assigned
     */
	boolean assignCourseToProfessor(int professorId, int courseId);
    /**
     * Adds a new professor to the system.
     *
     * @param professor the Professor object containing the details of the professor to be added
     */

	boolean approveProfessor(int professorId);
    /**
     * Retrieves a list of usernames of students whose registration is pending approval.
     *
     * @return a List of strings containing usernames of pending students
     */
    List<String> getPendingStudents();
    /**
     * Retrieves the course catalog.
     *
     * @return a List of Course objects representing the course catalog
     */
    List<Course> getCourseCatalog();
    /**
     * Approves a student's registration.
     *
     * @param username the username of the student to be approved
     */
	void approveStudent(String username);
	List<String> getPendingProfessor();
	List<Professor> getAllProfessors();
	boolean removeProfessor(int professorId) throws ProfessorRemovalException;
	boolean setBillAmount(int courseId, int newBillAmount);
	boolean removeProfessorFromCourse(int courseId);
}
