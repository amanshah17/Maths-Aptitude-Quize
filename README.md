# Maths-Aptitude-Quize
This Python script establishes a connection to a MySQL database and provides functionality for a simple quiz application with user registration and login. 
import mysql.connector
import bcrypt

# Database connection
mydb = mysql.connector.connect(
    host="localhost",
    user="root",
    password="Aman@1712",
    database="mydb"
)

# Function to fetch random questions
def fetch_random_questions():
    try:
        cursor = mydb.cursor(dictionary=True)
        cursor.execute("SELECT * FROM MathsAptitudeQuestions ORDER BY RAND() LIMIT 10")
        questions = cursor.fetchall()
        cursor.close()
        return questions
    except mysql.connector.Error as err:
        print("Error fetching questions:", err)
        return []

def check_answer(question_id, user_answer):
    try:
        cursor = mydb.cursor()
        cursor.execute("SELECT CorrectAnswer FROM MathsAptitudeQuestions WHERE QuestionID = %s", (question_id,))
        correct_option = cursor.fetchone()[0]
        cursor.close()
        return user_answer.upper() == correct_option
    except mysql.connector.Error as err:
        print("Error checking answer:", err)
        return False

def attempt_quiz():
    questions = fetch_random_questions()
    if not questions:
        print("No questions found. Unable to start quiz.")
        return

    score = 0

    for question in questions:
        print(f"\nQuestion: {question['QuestionText']}")
        print(f"A. {question['OptionA']}")
        print(f"B. {question['OptionB']}")
        print(f"C. {question['OptionC']}")
        print(f"D. {question['OptionD']}")

        user_answer = input("Enter your answer (A/B/C/D) or Q to quit: ").strip().upper()
        
        if user_answer == 'Q':
            print("You have chosen to exit the quiz.")
            break
        
        if check_answer(question['QuestionID'], user_answer):
            print("Correct!")
            score += 1
        else:
            print(f"Wrong! The correct answer was {question['CorrectAnswer']}.")

    print(f"\nYour total score: {score}/{len(questions)}")

# Function to register a new user
def register():
    cursor = mydb.cursor()
    name = input("Enter your name: ")

    while True:
        email_id = input("Enter your email: ")
        if "@gmail.com" in email_id:
            break
        else:
            print("Please enter a valid email address with @gmail.com extension.")

    while True:
        password = input("Enter your password: ")
        re_password = input("Enter your password again: ")
        if password == re_password:
            hashed_password = bcrypt.hashpw(password.encode('utf-8'), bcrypt.gensalt())
            print("Password created successfully")
            break
        else:
            print("Passwords do not match. Please re-enter password.")

    try:
        sql = 'INSERT INTO register(name, email_id, password) VALUES (%s, %s, %s)'
        val = (name, email_id, hashed_password.decode('utf-8'))
        cursor.execute(sql, val)
        mydb.commit()
        cursor.close()
        print(f"Hello {name}, your account has been created successfully")
    except mysql.connector.Error as err:
        print("Error registering:", err)

# Function to log in an existing user
def login():
    email_id = input("Enter your email: ")
    password = input("Enter your password: ")
    
    cursor = mydb.cursor()
    try:
        sql = "SELECT password FROM register WHERE email_id = %s"
        val = (email_id,)
        cursor.execute(sql, val)
        result = cursor.fetchone()
        cursor.close()
        
        if result and bcrypt.checkpw(password.encode('utf-8'), result[0].encode('utf-8')):
            print("Login successful! Welcome!")
            return True
        else:
            print("Invalid email or password.")
            return False
    except mysql.connector.Error as err:
        print("Error logging in:", err)
        return False

# Function to display the main menu
def main():
    while True:
        print("\nMenu:")
        print("1. Register")
        print("2. Login")
        print("3. Attempt Quiz")
        choice = input("Enter your choice: ")

        try:
            choice = int(choice)
        except ValueError:
            print("Invalid choice. Please enter a number.")
            continue
        
        if choice == 1:
            register()
        elif choice == 2:
            if login():
                print("You are now logged in.")
        elif choice == 3:
            print("Please login before attempting the quiz.")
            if login():
                attempt_quiz()
            break
        else:
            print("Invalid choice. Please enter a number between 1 and 3.")

# Function to close the application
def exit_app():
    print("Exiting the application.")
    mydb.close()

if __name__ == "__main__":
    main()
