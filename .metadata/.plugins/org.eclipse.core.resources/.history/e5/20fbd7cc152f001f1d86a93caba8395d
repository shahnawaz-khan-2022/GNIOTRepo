package com.gniot.crs.dao;

import java.util.List;

import com.gniot.crs.bean.Course;
import com.gniot.crs.bean.Student;

/**
 * Interface for Professor Data Access Object.
 */
/**
 * The ProfessorDAOInterface defines the data access methods that are required
 * for professor operations in the course registration system. These methods
 * will include SQL commands for interacting with the database to manage grades
 * and view enrolled students.
 */
public interface ProfessorDAOInterface {
	/**
	 * Adds a grade for a student in a specific course. This method will include SQL
	 * to insert or update the grade record in the database.
	 *
	 * @param studentId the ID of the student to whom the grade is assigned
	 * @param courseId  the ID of the course for which the grade is assigned
	 * @param grade     the grade to be assigned
	 * @return boolean indicating the success or failure of the operation
	 */
    void addGrade(int studentId, int courseId, String grade);

	/**
	 * Retrieves a list of student IDs enrolled in a specific course. This method
	 * will include SQL to query the database for enrolled students.
	 *
	 * @param courseId the ID of the course for which to view enrolled students
	 * @return a List of strings containing student IDs of enrolled students
	 */

	List<Student> getEnrolledStudents(int professorId);
	/**
	 * Adds a grade for a student in a specific course. This method includes SQL to
	 * insert or update the grade record in the database.
	 *
	 * @param studentId the ID of the student to whom the grade is assigned
	 * @param courseId  the ID of the course for which the grade is assigned
	 * @param grade     the grade to be assigned
	 * @return boolean indicating the success or failure of the operation
	 */
	List<Course> getProfessorCourses(int professorId);

    List<Student> getEnrolledStudentsInCourse(int courseId);

	int currentProfessorId();
	

	/**
	 * Adds a grade for a student in a specific course. This method includes SQL to
	 * insert or update the grade record in the database.
	 *
	 * @param studentId the ID of the student to whom the grade is assigned
	 * @param courseId  the ID of the course for which the grade is assigned
	 * @param grade     the grade to be assigned
	 * @return boolean indicating the success or failure of the operation
	 */

}
