# DWM-
import sqlite3
import pandas as pd


conn = sqlite3.connect("education_dw.db")
cursor = conn.cursor()


cursor.execute("""
CREATE TABLE IF NOT EXISTS dim_student (
    student_id INTEGER PRIMARY KEY,
    student_name TEXT,
    gender TEXT,
    department TEXT
);
""")

cursor.execute("""
CREATE TABLE IF NOT EXISTS dim_course (
    course_id INTEGER PRIMARY KEY,
    course_name TEXT,
    credits INTEGER,
    course_type TEXT
);
""")

cursor.execute("""
CREATE TABLE IF NOT EXISTS dim_instructor (
    instructor_id INTEGER PRIMARY KEY,
    instructor_name TEXT,
    qualification TEXT,
    experience INTEGER
);
""")

cursor.execute("""
CREATE TABLE IF NOT EXISTS dim_time (
    time_id INTEGER PRIMARY KEY,
    semester TEXT,
    academic_year TEXT
);
""")



cursor.execute("""
CREATE TABLE IF NOT EXISTS fact_student_performance (
    performance_id INTEGER PRIMARY KEY,
    student_id INTEGER,
    course_id INTEGER,
    instructor_id INTEGER,
    time_id INTEGER,
    marks_obtained REAL,
    grade TEXT,

    FOREIGN KEY (student_id) REFERENCES dim_student(student_id),
    FOREIGN KEY (course_id) REFERENCES dim_course(course_id),
    FOREIGN KEY (instructor_id) REFERENCES dim_instructor(instructor_id),
    FOREIGN KEY (time_id) REFERENCES dim_time(time_id)
);
""")



cursor.executemany(
    "INSERT OR REPLACE INTO dim_student VALUES (?, ?, ?, ?);",
    [
        (1, 'Riya Sharma', 'Female', 'Computer Science'),
        (2, 'Amit Patel', 'Male', 'Information Technology'),
        (3, 'Neha Gupta', 'Female', 'Electronics'),
        (4, 'Rahul Mehta', 'Male', 'Computer Science')
    ]
)

cursor.executemany(
    "INSERT OR REPLACE INTO dim_course VALUES (?, ?, ?, ?);",
    [
        (101, 'Database Systems', 4, 'Core'),
        (102, 'Data Mining', 3, 'Elective'),
        (103, 'Operating Systems', 4, 'Core')
    ]
)

cursor.executemany(
    "INSERT OR REPLACE INTO dim_instructor VALUES (?, ?, ?, ?);",
    [
        (201, 'Dr. Sunita Desai', 'PhD (Computer Science)', 12),
        (202, 'Prof. Arjun Rao', 'M.Tech (IT)', 8),
        (203, 'Dr. Meena Joshi', 'PhD (Electronics)', 10)
    ]
)

cursor.executemany(
    "INSERT OR REPLACE INTO dim_time VALUES (?, ?, ?);",
    [
        (301, 'Sem-I', '2023-24'),
        (302, 'Sem-II', '2023-24')
    ]
)

cursor.executemany(
    "INSERT OR REPLACE INTO fact_student_performance VALUES (?, ?, ?, ?, ?, ?, ?);",
    [
        (1, 1, 101, 201, 301, 89.5, 'A'),
        (2, 2, 101, 201, 301, 78.0, 'B'),
        (3, 3, 103, 203, 302, 85.25, 'A'),
        (4, 4, 102, 202, 302, 90.0, 'A+'),
        (5, 1, 102, 202, 302, 82.0, 'A')
    ]
)

conn.commit()


print("\n--- Average Marks per Course (Roll-Up) ---")

print(pd.read_sql_query("""
SELECT c.course_name, AVG(f.marks_obtained) AS avg_marks
FROM fact_student_performance f
JOIN dim_course c ON f.course_id = c.course_id
GROUP BY c.course_name;
""", conn))


print("\n--- Drill-Down by Department and Course ---")

print(pd.read_sql_query("""
SELECT s.department, c.course_name, AVG(f.marks_obtained) AS avg_marks
FROM fact_student_performance f
JOIN dim_student s ON f.student_id = s.student_id
JOIN dim_course c ON f.course_id = c.course_id
GROUP BY s.department, c.course_name
ORDER BY s.department;
""", conn))


conn.close()
