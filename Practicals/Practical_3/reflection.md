# Practical 3 Reflection - Class and Object Diagrams for Automated Grading System

## 1. Overview

This practical involved creating UML Class and Object diagrams based on the Automated Grading System designed in Practical 2. The goal was to translate the use cases and interaction overview diagrams into a detailed structural model showing how the system components relate to each other and interact through specific instances.

---

## 2. Class Diagram Analysis

### 2.1 Core Classes

The class diagram identifies nine key classes that make up the Automated Grading System:

**User-Facing Classes:**
- **Student**: Represents students submitting assignments with attributes like studentID, name, and email. Methods include submitAssignment() and viewGrade().
- **Professor**: Represents faculty members who review submissions and can override grades. Methods include reviewSubmission(), overrideGrade(), and viewReport().

**Assignment & Submission Classes:**
- **Assignment**: Models the assignment itself with metadata (assignmentID, title, dueDate). Contains getSubmissions() and checkDeadline() methods.
- **Submission**: Represents individual student submissions with timestamp, sourceCode, and status. Includes runCode() and getStatus() methods.

**Grading Classes:**
- **Grade**: Represents the awarded grade with score, comments, and gradeID. Includes calculateScore() and recordGrade() methods.

**System Processing Classes:**
- **GradingSystem**: The central orchestrator that processes submissions, executes code, generates reports, and syncs with the LMS. This is the "middleman" component that automates the manual professor work described in Practical 2.
- **PlagiarismChecker**: A specialized component for plagiarism detection with methods checkInternalSimilarity() and checkTurnItIn(). Uses a threshold attribute to determine when plagiarism is flagged.
- **TurnItInAPI**: External API integration for cloud-based plagiarism checking with apiKey, endpoint, and timeout attributes.
- **LMSConnector**: Handles synchronization with the university's mainframe LMS. Methods include syncGrades(), pushToLMS(), and verifyConnection().

### 2.2 Relationships and Dependencies

The class diagram reveals several important structural relationships:

1. **Submission Flow**: Student → Assignment → Submission → Grade. This represents the core workflow where students submit to assignments, which are then processed into submissions and receive grades.

2. **Automation Chain**: GradingSystem processes Submission and generates Grade. The system acts as the central hub automating all operations.

3. **Quality Assurance**: PlagiarismChecker checks Submission in parallel with grade calculation. This ensures academic integrity while the system assigns scores.

4. **External Integration**: PlagiarismChecker uses TurnItInAPI, representing the parallel plagiarism detection approach described in Practical 2 where submissions are checked both internally and through TurnItIn.

5. **LMS Synchronization**: GradingSystem uses LMSConnector, addressing the constraint that the university's mainframe LMS cannot be directly modified. The system must push data to it instead.

6. **Human Oversight**: Professor reviews/overrides Grade, maintaining the human element for edge cases and unusual situations.

---

## 3. Object Diagram Analysis

### 3.1 Instance Representation

The object diagram brings the class model to life with concrete instances representing a typical grading scenario:

**Specific Objects:**
- `john:Student` - A student with ID S001 submitting work
- `assign1:Assignment` - A "Data Structures" assignment due April 15, 2026
- `sub001:Submission` - John's submission processed on April 10, 2026
- `grade001:Grade` - A score of 87.5 with comments "Good work"
- `gradeSystem:GradingSystem` - Managing multiple submissions and grades
- `plagChecker:PlagiarismChecker` - Checking for plagiarism with 25% threshold
- `turnItIn:TurnItInAPI` - Connected to the TurnItIn service
- `lmsConn:LMSConnector` - Currently connected to the LMS
- `prof:Professor` - Dr. Jane Doe, who can review and override grades

### 3.2 Instance Relationships

The object diagram shows how these specific instances interact:

1. **Submission Workflow**: John submits to the Data Structures assignment, which contains his submission that receives an 87.5 grade.

2. **System Processing**: The GradingSystem processes John's submission and generates his grade, while the PlagiarismChecker examines the submission through both internal checking and TurnItIn.

3. **Synchronization**: The LMSConnector pushes the grade data to the university's LMS, completing the automated workflow.

4. **Human Review**: Professor Doe can review John's grade (87.5) and override it if needed based on the flagged issues or circumstances.

---

## 4. Relationship Between Class and Object Diagrams

The class diagram provides the **blueprint** while the object diagram provides the **instance**. Key observations:

- **One-to-Many**: The class diagram shows one Assignment can contain multiple Submissions (implied by getSubmissions()). The object diagram shows one example where assign1 contains sub001.

- **Collections**: GradingSystem maintains lists of submissions and grades. In the object diagram, gradeSystem shows `submissions = [sub001, sub002]` and `grades = [grade001, grade002]`, illustrating that the system handles multiple students simultaneously.

- **Optional Relationships**: While the class diagram shows all possible relationships, the object diagram shows only the active relationships for this specific scenario. Professor review may not always happen if the grade is within normal ranges.

---

## 5. Design Insights

### 5.1 Addressing Practical 2 Requirements

The diagrams successfully translate the business requirements from Practical 2:

1. **Automation**: The GradingSystem class with its processSubmission() and executeCode() methods demonstrates how the manual professor work is now automated.

2. **Plagiarism Detection**: The dual approach (internal + TurnItIn) is modeled in PlagiarismChecker, which satisfies the requirement for comprehensive plagiarism checking.

3. **LMS Integration**: LMSConnector addresses the constraint that the mainframe LMS cannot be modified. The system must sync with it rather than replace it.

4. **Cost Efficiency**: Using TurnItInAPI's existing service instead of building custom plagiarism detection aligns with the limited IT budget constraint mentioned in Practical 2.

5. **Academic Integrity**: The Professor class with overrideGrade() method ensures human judgment is maintained for edge cases, preserving academic standards.

### 5.2 Architectural Patterns

The class diagram reveals several design patterns:

- **Facade Pattern**: GradingSystem acts as a facade, providing a single interface (processSubmission()) that internally coordinates multiple components (PlagiarismChecker, Grade generation, LMS sync).

- **Adapter Pattern**: LMSConnector adapts the internal grading system to the external mainframe LMS interface.

- **Separation of Concerns**: Each class has a clear, focused responsibility (Student submits, GradingSystem processes, PlagiarismChecker checks, LMSConnector syncs).

---

## 6. Consistency with Practical 2

The diagrams maintain consistency with earlier UML deliverables:

1. **Actors from IoD/UCD**: Student, Professor, and indirectly TurnItIn (external system) align with actors identified in Practical 2.

2. **Use Cases as Methods**: Use cases from Practical 2 like "Submit Assignment," "Run Code," "Calculate Grade," "Check Plagiarism," and "Sync with LMS" appear as methods in the respective classes.

3. **Business Outcome**: Both diagrams support the business outcome "Assignment Successfully Graded and Recorded in LMS" through the workflow from Submission to Grade to LMS synchronization.

---

## 7. Potential Improvements

If this design were to be extended further:

1. **Exception Handling**: Add an ErrorHandler or NotificationSystem class to manage scenarios where TurnItIn API is down or LMS sync fails (as I mentioned in Practical 2's reflection).

2. **Audit Trail**: A GradeAuditLog class to track grade overrides by professors, supporting academic integrity and compliance requirements.

3. **Submission Versioning**: Track multiple submission attempts separately to support the "Resubmit Assignment" use case from Practical 2.

4. **Notification System**: A Notifier class to handle alerts to students and professors when processing is complete or plagiarism is detected.

5. **Code Execution Sandbox**: A CodeExecutor class to safely run student code in an isolated environment, preventing system compromise.

6. **Caching and Performance**: A Cache layer for frequently accessed data (assignment metadata, previously calculated grades) to improve system responsiveness.

---

## 8. Conclusion

The Class and Object diagrams provide a clear structural view of the Automated Grading System. The class diagram shows the system's architecture with well-defined responsibilities for each component, while the object diagram demonstrates how these components work together through a concrete example.

Together, these diagrams bridge the gap between the high-level business requirements (from Practical 2's interaction overview) and the detailed implementation specifications (which would follow in design patterns and code). They show not just *what* the system needs to do (use cases), but *how* it's organized to do it (structural components and their interactions).

The design successfully addresses all the constraints mentioned in Practical 2: the unmovable mainframe LMS, the limited IT budget (using TurnItIn's API), and the need for academic integrity (plagiarism checking + professor oversight). This demonstrates that good system design requires understanding both the business requirements and the real-world constraints under which the system must operate.

---

## 9. References

- Practical 2: Use Case Diagram and Interaction Overview Diagrams for Automated Grading System
- UML Specifications: Class Diagrams and Object Diagrams
- System Design Principles: Separation of Concerns, Facade Pattern, Adapter Pattern
