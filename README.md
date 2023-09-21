# Project_1
mport sqlite3
import json
import requests
import xml.etree.ElementTree as ET

try:
    # Creates or opens a file called HyperionDev with a SQLite3 DB. 
    conn = sqlite3.connect("HyperionDev.db")

except sqlite3.Error:
    print("Please store your database as HyperionDev.db")
    quit()

cur = conn.cursor() # Get a cursor object.

def usage_is_incorrect(input, num_args):
    if len(input) != num_args + 1:
        print(f"The {input[0]} command requires {num_args} arguments.")
        return True
    return False

def store_data_as_json(data):
    resulting_query = requests.get(data)
    # Save the data in a json file.
    with open('data.json', 'w', encoding='utf-8') as outfile:
        outfile = json.dump(resulting_query.json(), outfile, sort_keys=True, indent=4)

def store_data_as_xml(data):
    # Create the root element
    root = ET.parse('filename.xml')
    # Create the XML tree with the root element
    tree = ET.ElementTree(root)
    # Write the XML tree to the file
    tree.write(data, xml_declaration=True, encoding='utf-8')
    
def offer_to_store(data):

    while True:
        print("Would you like to store this result?")
        choice = input("Y/[N]? : ").strip().lower()

        if choice == "y":
            filename = input("Specify filename. Must end in .xml or .json: ")
            ext = filename.split(".")[-1]
            if ext == 'xml':
                store_data_as_xml(data)
            elif ext == 'json':
                store_data_as_json(data)
            else:
                print("Invalid file extension. Please use .xml or .json")

        elif choice == 'n':
            break

        else:
            print("Invalid choice")

usage = '''
What would you like to do?

d - demo
vs <student_id>            - view subjects taken by a student
la <firstname> <surname>   - lookup address for a given firstname and surname
lr <student_id>            - list reviews for a given student_id
lc <teacher_id>            - list all courses taken by teacher_id
lnc                        - list all students who haven't completed their course
lf                         - list all students who have completed their course and achieved 30 or below
e                          - exit this program

Type your option here: '''

print("Welcome to the data querying app!")

while True:
    print()
    # Get input from user
    user_input = input(usage).split(" ")
    print()

    # Parse user input into command and args
    command = user_input[0]
    if len(user_input) > 1:
        args = user_input[1:]

    if command == 'd': # demo - a nice bit of code from me to you - this prints all student names and surnames :)
        data = cur.execute("SELECT * FROM Student")
        for _, firstname, surname, _, _ in data:
            print(f"{firstname} {surname}")
        
    elif command == 'vs': 
        '''
        View all subjects being taken by a specified student (search by
        student_id). On console, the subject name should only be shown
        '''
        if usage_is_incorrect(user_input, 1):
            continue
        student_id = args[0]
        cur.execute('''SELECT subjects 
                       FROM Student
                       WHERE student_id=?''', (student_id,))
        data = cur.fetchone()
        for subjects in data:
                print(subjects)

        offer_to_store(data)

    elif command == 'la':
        '''
        Look up an address given a first name and a surname.
        Only the street name and city should be shown on the console
        '''
        if usage_is_incorrect(user_input, 2):
            continue
        firstname, surname = args[0], args[1]
        cur.execute('''SELECT street name, city 
                       FROM Student 
                       WHERE firstname=? AND surname=?''', (firstname, surname))
        data = cur.fetchall()
        if data:
            for row in data:
                stree_name = row[0]
                city = row[1]
                print(f'The street name is {stree_name}')
                print(f'The city is {city}')

            offer_to_store(data)
        else:
            print(f"No street name nor city were found for {firstname} {surname}")
            continue
    
    elif command == 'lr':
        '''
        List all reviews given to a student (search by student_id).
        The completeness, efficiency, style and documentation scores
        should be displayed on console, along with the review text.
        '''
        if usage_is_incorrect(user_input, 1):
            continue
        student_id = args[0]
        cur.execute(f'''SELECT completeness, efficiency, style, documentation, review
                        FROM Review
                        WHERE Student_id = {student_id}''')
        data = cur.fetchall()
        if len(data) != 0:
             for row in data:
                completenss = row[0]
                efficiency = row[1]
                style = row[2]
                documentation = row[3]
                review = row[4]
                print(f'''The student with ID number {student_id} has achieved the following scores:
                       {completeness}, {efficiency}, {style}, {documentation}, {review}.''')
    
             offer_to_store(data)
        else:
            print(f"No reviews for", student_id, "were found!")
            continue

    elif command == 'lc':
        '''
        List all courses being given by a specific teacher (search by teacher_id).
        Just the course name should be displayed on the console..
        '''
        if usage_is_incorrect(user_input, 1):
            continue
        teacher_id = args[0]
        cur.execute('''SELECT courses 
                       FROM Student 
                       WHERE teacher_id=?''', (teacher_id,))
        data = cur.fetchall()
        for courses in data:
                print(courses)

        offer_to_store(data)
    
    elif command == 'lnc':
        '''
        List all students who haven’t completed their course.
        The student number, first and last names, email addresses and
        course names should be shown on the console.
        '''
        if usage_is_incorrect(user_input, 1):
            continue
        completeness = args[0]
        cur.execute('''SELECT student_number, firstname, surname, email_address,course_name 
                       FROM Student 
                       WHERE completeness= 'No' ''', (completeness,))
        data = cur.fetchall()
        if data:
            for row in data:
                student_number = row[0]
                firstname = row[1]
                surname = row[2]
                email_address = row[3]
                course_name = row[4]
                print(f'''The following{student_number}, {firstname}, {surname}, 
                    {email_address}, {course_name} belong to the student(s) who has/have not completed their course''')
            offer_to_store(data)
        else:
            print('All students have completed their course')
    
    elif command == 'lf':
        '''
        List all students who have completed their course and achieved a mark of 30 or below.
        The student number, first and last names, email addresses andcourse names should be shown on the console. 
        Their marks should also be displayed
        '''
        if usage_is_incorrect(user_input, 1):
            continue
        completeness = args[0]
        cur.execute('''SELECT student_number, firstname, surname, email_address, course_name, marks 
                       FROM Student 
                       WHERE completeness= 'Yes' AND marks <= 30 ''', (completeness,))
        data = cur.fetchall()
        if data:
            for row in data:
                student_number = row[0]
                firstname = row[1]
                surname = row[2]
                email_address = row[3]
                course_name = row[4]
                marks = row[5]
                print(f'''The following{student_number}, {firstname}, {surname}, 
                    {email_address}, {course_name} belong to the student(s) who has/have completed 
                    their course with the following mark: {marks}''')
            
            offer_to_store(data)
    
    elif command == 'e':# list address by name and surname
        print("Programme exited successfully!")
        break
    
    else:
        print(f"Incorrect command: '{command}'")
    

    
