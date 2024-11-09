# 🧠 Quiz App
This Quiz App fetches questions from the Open Trivia Database API, presents them in a True/False format, and tracks the player’s score. It uses Tkinter for a clean and interactive GUI.

## 📂 Project Structure
- **data.py**: Fetches questions from the Open Trivia Database API.
- **main.py**: Initializes the application by creating question instances, the quiz brain, and the UI interface.
- **question_model.py**: Defines the Question class.
- **quiz_brain.py**: Contains logic for handling questions, score, and checking answers.
- **ui.py**: Manages the user interface, including displaying questions and tracking scores.
---

## 🔧 Code Explanation
### 1. data.py: Fetch Questions from API
```python
import requests

parameters = {
    "amount": 10,
    "type": "boolean",
    "category": 18,
}

response = requests.get("https://opentdb.com/api.php", params=parameters)
response.raise_for_status()
data = response.json()
questionData = data["results"]
```
- **parameters**: Sets the number, type, and category of questions.
- **questionData**: Stores the question data fetched from the API in JSON format.
### 2. main.py: Initialize Quiz
```python
from question_model import Question
from data import questionData
from quiz_brain import QuizBrain
from ui import QuizInterface

question_bank = [Question(q["question"], q["correct_answer"]) for q in questionData]

quiz = QuizBrain(question_bank)
quizUi = QuizInterface(quiz)

print("You've completed the quiz")
print(f"Your final score was: {quiz.score}/{quiz.question_number}")
```
- **question_bank**: Creates a list of Question objects from the API data.
- **QuizInterface**: Launches the UI with QuizBrain handling the quiz logic.
### 3. question_model.py: Define Question Class
```python
class Question:
    def __init__(self, q_text, q_answer):
        self.text = q_text
        self.answer = q_answer
```
- **Question**: Holds the text and answer for each question.
### 4. quiz_brain.py: Quiz Logic
```python
import html

class QuizBrain:
    def __init__(self, q_list):
        self.question_number = 0
        self.score = 0
        self.question_list = q_list
        self.current_question = None

    def still_has_questions(self):
        return self.question_number < len(self.question_list)

    def next_question(self):
        self.current_question = self.question_list[self.question_number]
        self.question_number += 1
        qTEXT = html.unescape(self.current_question.text)
        return f"Q.{self.question_number}: {qTEXT}"

    def check_answer(self, user_answer):
        correct_answer = self.current_question.answer
        if user_answer.lower() == correct_answer.lower():
            self.score += 1
            return True
        else:
            return False
```
- **next_question**: Returns the current question text, formatted for display.
- **check_answer**: Checks if the user’s answer is correct and updates the score.
### 5. ui.py: User Interface
```python
import time
from tkinter import *
from quiz_brain import QuizBrain

THEME_COLOR = "#375362"

class QuizInterface:
    def __init__(self, quizBrain: QuizBrain):
        self.quiz = quizBrain
        self.window = Tk()
        self.window.title("Quizzler")
        self.window.config(padx=20, pady=20, bg=THEME_COLOR)

        self.scoreLabel = Label(text="Score: 0", fg="white", bg=THEME_COLOR)
        self.scoreLabel.grid(row=0, column=1)
        
        self.canvas = Canvas(width=300, height=250, bg="white")
        self.questionText = self.canvas.create_text(150, 125, width=280, text="Some Question Text", fill=THEME_COLOR, font=("Arial", 20, "italic"))
        self.canvas.grid(row=1, column=0, columnspan=2, pady=50)
        
        trueImage = PhotoImage(file="images/true.png")
        self.trueButton = Button(image=trueImage, highlightthickness=0, command=self.truePressed)
        self.trueButton.grid(row=2, column=0)

        falseImage = PhotoImage(file="images/false.png")
        self.falseButton = Button(image=falseImage, highlightthickness=0, command=self.falsePressed)
        self.falseButton.grid(row=2, column=1)

        self.getNextQuestion()
        self.window.mainloop()

    def getNextQuestion(self):
        self.canvas.config(bg="white")
        if self.quiz.still_has_questions():
            self.scoreLabel.config(text=f"Score: {self.quiz.score}")
            qText = self.quiz.next_question()
            self.canvas.itemconfig(self.questionText, text=qText)
        else:
            self.canvas.itemconfig(self.questionText, text="You've reached the end of the quiz.")
            self.trueButton.config(state="disabled")
            self.falseButton.config(state="disabled")

    def truePressed(self):
        self.giveFeedback(self.quiz.check_answer("True"))

    def falsePressed(self):
        isRight = self.quiz.check_answer("False")
        self.giveFeedback(isRight)

    def giveFeedback(self, isRight):
        self.canvas.config(bg="green" if isRight else "red")
        self.window.after(1000, self.getNextQuestion)
```
- **QuizInterface**: Manages GUI components, question display, and user interaction.
- **giveFeedback**: Changes canvas color based on correct or incorrect answer and shows the next question after a delay.
---
## 🚀 How to Run
1. Install Required Libraries: Ensure requests is installed.
```bash
pip install requests
```
2. Images: Place true.png and false.png in an images folder in the same directory as the script.
3. Run the Application:
```bash
python main.py
```
4. Answer Questions: Click the True or False button to answer. The app will display your score once you finish.

---
