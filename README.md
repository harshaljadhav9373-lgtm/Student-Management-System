# Student-Management-System
#include <iostream>
#include <fstream>
#include <string>
#include <iomanip>
#include <vector>

using namespace std;

// Class representing a individual Student
class Student {
private:
    int rollNumber;
    string name;
    string grade;
    int age;

public:
    // Constructor
    Student() : rollNumber(0), age(0) {}
    Student(int roll, string n, int a, string g) : rollNumber(roll), name(n), age(a), grade(g) {}

    // Getters
    int getRollNumber() const { return rollNumber; }
    string getName() const { return name; }
    int getAge() const { return age; }
    string getGrade() const { return grade; }

    // Setters for updating data
    void setName(string n) { name = n; }
    void setAge(int a) { age = a; }
    void setGrade(string g) { grade = g; }

    // Display student record in a formatted row
    void displayRow() const {
        cout << left << setw(15) << rollNumber 
             << setw(25) << name 
             << setw(10) << age 
             << setw(10) << grade << endl;
    }
};

// Main System Class to manage File Operations
class StudentSystem {
private:
    const string filename = "students.txt";

    // Helper method to load all students from the file into a memory vector
    vector<Student> loadAllStudents() {
        vector<Student> students;
        ifstream file(filename);
        if (!file.is_open()) {
            return students; // Return empty if file doesn't exist yet
        }

        int roll, age;
        string name, grade;
        
        // Reading space/newline separated data safely
        while (file >> roll) {
            file.ignore(); // clear the separator
            getline(file, name, '|'); // Read name until the pipe delimiter
            file >> age >> grade;
            students.push_back(Student(roll, name, age, grade));
        }
        file.close();
        return students;
    }

    // Helper method to overwrite the file with the updated vector list
    void saveAllStudents(const vector<Student>& students) {
        ofstream file(filename, ios::trunc); // Overwrite mode
        if (!file.is_open()) {
            cout << "\tError: Could not open file to save data.\n";
            return;
        }

        for (const auto& student : students) {
            // Using '|' delimiter to allow spaces in student names safely
            file << student.getRollNumber() << " " 
                 << student.getName() << "|" 
                 << student.getAge() << " " 
                 << student.getGrade() << "\n";
        }
        file.close();
    }

public:
    // 1. ADD NEW STUDENT
    void addStudent() {
        vector<Student> students = loadAllStudents();
        int roll, age;
        string name, grade;

        cout << "\n================= ADD NEW STUDENT =================" << endl;
        cout << "\tEnter Roll Number: ";
        while (!(cin >> roll)) {
            cout << "\tInvalid input. Enter a numeric Roll Number: ";
            cin.clear();
            cin.ignore(123, '\n');
        }

        // Check for duplicate Roll Number
        for (const auto& s : students) {
            if (s.getRollNumber() == roll) {
                cout << "\tError: Student with Roll Number " << roll << " already exists!\n";
                return;
            }
        }

        cin.ignore(); // Clear input buffer
        cout << "\tEnter Student Name: ";
        getline(cin, name);
        
        cout << "\tEnter Age: ";
        while (!(cin >> age) || age <= 0) {
            cout << "\tInvalid input. Enter a valid age: ";
            cin.clear();
            cin.ignore(123, '\n');
        }

        cout << "\tEnter Grade/Class: ";
        cin >> grade;

        // Add to vector and save back to file
        students.push_back(Student(roll, name, age, grade));
        saveAllStudents(students);
        cout << "\n\t✔ Student record added successfully!\n";
    }

    // 2. DISPLAY ALL STUDENTS
    void displayAll() {
        vector<Student> students = loadAllStudents();

        cout << "\n=======================================================\n";
        cout << "\t\tSTUDENT RECORDS LIST\n";
        cout << "=======================================================\n";
        
        if (students.empty()) {
            cout << "\t\tNo records found!\n";
            cout << "=======================================================\n";
            return;
        }

        cout << left << setw(15) << "Roll No" 
             << setw(25) << "Name" 
             << setw(10) << "Age" 
             << setw(10) << "Grade" << endl;
        cout << "-------------------------------------------------------\n";

        for (const auto& student : students) {
            student.displayRow();
        }
        cout << "=======================================================\n";
    }

    // 3. UPDATE STUDENT RECORD
    void updateStudent() {
        vector<Student> students = loadAllStudents();
        int roll;
        bool found = false;

        cout << "\n================= UPDATE STUDENT =================\n";
        cout << "\tEnter Roll Number to update: ";
        cin >> roll;

        for (auto& student : students) {
            if (student.getRollNumber() == roll) {
                found = true;
                string newName, newGrade;
                int newAge;

                cout << "\n\tRecord Found! Current Details:\n";
                cout << "\tName: " << student.getName() << " | Age: " << student.getAge() << " | Grade: " << student.getGrade() << "\n\n";

                cin.ignore();
                cout << "\tEnter New Name (or press Enter to keep current): ";
                getline(cin, newName);
                if (!newName.empty()) student.setName(newName);

                cout << "\tEnter New Age (or enter 0 to keep current): ";
                while (!(cin >> newAge) || newAge < 0) {
                    cout << "\tInvalid input. Enter a valid age: ";
                    cin.clear();
                    cin.ignore(123, '\n');
                }
                if (newAge != 0) student.setAge(newAge);

                cout << "\tEnter New Grade (or enter '-' to keep current): ";
                cin >> newGrade;
                if (newGrade != "-") student.setGrade(newGrade);

                break;
            }
        }

        if (found) {
            saveAllStudents(students);
            cout << "\n\t✔ Student record updated successfully!\n";
        } else {
            cout << "\t✖ Error: Student with Roll Number " << roll << " not found.\n";
        }
    }

    // 4. DELETE STUDENT RECORD
    void deleteStudent() {
        vector<Student> students = loadAllStudents();
        int roll;
        bool found = false;

        cout << "\n================= DELETE STUDENT =================\n";
        cout << "\tEnter Roll Number to delete: ";
        cin >> roll;

        // Iterator to look for the matching roll number
        for (auto it = students.begin(); it != students.end(); ++it) {
            if (it->getRollNumber() == roll) {
                students.erase(it);
                found = true;
                break;
            }
        }

        if (found) {
            saveAllStudents(students);
            cout << "\n\t✔ Student record deleted successfully!\n";
        } else {
            cout << "\t✖ Error: Student with Roll Number " << roll << " not found.\n";
        }
    }
};

// Main Driver Function with Menu Interface
int main() {
    StudentSystem system;
    int choice;

    do {
        cout << "\n==========================================\n";
        cout << "        STUDENT MANAGEMENT SYSTEM         \n";
        cout << "==========================================\n";
        cout << "\t1. Add Student Record\n";
        cout << "\t2. Display All Student Records\n";
        cout << "\t3. Update Student Record\n";
        cout << "\t4. Delete Student Record\n";
        cout << "\t5. Exit Application\n";
        cout << "------------------------------------------\n";
        cout << "Enter your choice (1-5): ";
        
        // Validating menu input integer
        if (!(cin >> choice)) {
            cout << "\tInvalid choice! Please enter a number between 1 and 5.\n";
            cin.clear();
            cin.ignore(123, '\n');
            continue;
        }

        switch (choice) {
            case 1:
                system.addStudent();
                break;
            case 2:
                system.displayAll();
                break;
            case 3:
                system.updateStudent();
                break;
            case 4:
                system.deleteStudent();
                break;
            case 5:
                cout << "\n\tThank you for using the Student Management System. Goodbye!\n";
                break;
            default:
                cout << "\tInvalid option selection. Please try again.\n";
        }
    } while (choice != 5);

    return 0;
}
