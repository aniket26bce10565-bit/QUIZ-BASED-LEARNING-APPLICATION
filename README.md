# QUIZ-BASED-LEARNING-APPLICATION
The Python Quiz-Based Learning Application is an interactive educational software program designed to facilitate self-assessment and knowledge reinforcement. Built using Python, this application serves as an engaging platform where users can select educational topics, answer multiple-choice questions, receive instant feedback,.
#!/usr/bin/env python3
"""
Python Quiz Learning App
------------------------
A command-line, quiz-based application for learning Python.

Features
- Topics: Basics, Strings, Lists, Dictionaries, Functions, OOP
- Difficulty levels: easy / medium / hard
- Question types: multiple choice, predict-the-output, fill-in-the-blank
- Hints (cost half the points), explanations after every answer
- Streak bonus, final grade, and per-topic progress saved to a JSON file

Run:  python python_quiz_app.py
"""

import json
import random
import time
from dataclasses import dataclass, field
from pathlib import Path
from typing import List, Optional

PROGRESS_FILE = Path.home() / ".python_quiz_progress.json"
POINTS_PER_QUESTION = 10
HINT_PENALTY = 5
STREAK_BONUS = 2  # extra points per consecutive correct answer beyond the first


# --------------------------------------------------------------------------
# Data model
# --------------------------------------------------------------------------
@dataclass
class Question:
    topic: str
    level: str                 # easy | medium | hard
    kind: str                  # mcq | output | fill
    text: str
    answer: List[str]          # accepted answers (lower-case for text kinds)
    explanation: str
    hint: str = ""
    options: List[str] = field(default_factory=list)  # only for mcq


def mcq(topic, level, text, options, correct_index, explanation, hint=""):
    return Question(topic, level, "mcq", text, [chr(65 + correct_index).lower()],
                    explanation, hint, options)


def output(topic, level, code, answer, explanation, hint=""):
    return Question(topic, level, "output",
                    "What is the output of this code?\n\n" + code,
                    [a.lower() for a in answer], explanation, hint)


def fill(topic, level, text, answer, explanation, hint=""):
    return Question(topic, level, "fill", text,
                    [a.lower() for a in answer], explanation, hint)


# --------------------------------------------------------------------------
# Question bank  (add your own here!)
# --------------------------------------------------------------------------
QUESTIONS: List[Question] = [
    # ---- Basics ----
    mcq("Basics", "easy", "Which function prints text to the screen?",
        ["echo()", "print()", "display()", "write()"], 1,
        "print() sends output to standard output (the console).",
        "It has the same name as what it does on a printer."),
    mcq("Basics", "easy", "Which of these is a valid variable name?",
        ["2name", "my-name", "my_name", "class"], 2,
        "Names can't start with a digit, contain '-', or be a keyword like 'class'.",
        "Underscores are allowed; hyphens are not."),
    fill("Basics", "easy", "Fill in the blank: the type of the value 3.14 is ______ .",
         ["float"], "Numbers with a decimal point are floats.",
         "Not int, not decimal."),
    output("Basics", "medium", "print(7 // 2)", ["3"],
           "// is floor division: 7 / 2 = 3.5, floored to 3."),
    output("Basics", "medium", "print(2 ** 3 ** 2)", ["512"],
           "** is right-associative: 3 ** 2 = 9, then 2 ** 9 = 512.",
           "Evaluate the exponent on the right first."),
    mcq("Basics", "hard", "What does bool([]) return?",
        ["True", "False", "None", "Raises an error"], 1,
        "Empty containers are 'falsy', so bool([]) is False."),

    # ---- Strings ----
    output("Strings", "easy", "print('python'.upper())", ["python".upper()],
           "upper() returns a new string in uppercase."),
    fill("Strings", "easy", "Which string method removes whitespace from both ends? (name only)",
         ["strip", "strip()"], "strip() removes leading and trailing whitespace.",
         "Starts with 's'."),
    output("Strings", "medium", "s = 'hello'\nprint(s[1:4])", ["ell"],
           "Slicing [1:4] takes indexes 1, 2, 3 (end index excluded).",
           "Index 0 is 'h'."),
    output("Strings", "medium", "print('a,b,c'.split(','))", ["['a', 'b', 'c']"],
           "split() returns a list of the pieces."),
    output("Strings", "hard", "print('abc'[::-1])", ["cba"],
           "A step of -1 walks the string backwards.",
           "Think: reverse."),
    mcq("Strings", "hard", "Are Python strings mutable?",
        ["Yes", "No", "Only f-strings", "Only if empty"], 1,
        "Strings are immutable; operations return new strings."),

    # ---- Lists ----
    output("Lists", "easy", "nums = [1, 2, 3]\nnums.append(4)\nprint(len(nums))", ["4"],
           "append() adds one element, so the length becomes 4."),
    fill("Lists", "easy", "Which list method adds an item at the END? (name only)",
         ["append", "append()"], "append(x) adds x to the end of the list."),
    output("Lists", "medium", "print([x * x for x in range(4)])", ["[0, 1, 4, 9]"],
           "List comprehension squares 0, 1, 2, 3.",
           "range(4) gives 0..3."),
    output("Lists", "medium", "a = [3, 1, 2]\nprint(sorted(a))\nprint(a)",
           ["[1, 2, 3]\n[3, 1, 2]"],
           "sorted() returns a new list; the original stays unchanged."),
    mcq("Lists", "hard", "What is the output of: a = [1, 2]; b = a; b.append(3); print(a)?",
        ["[1, 2]", "[1, 2, 3]", "[3]", "Error"], 1,
        "b and a reference the SAME list object, so mutating b changes a.",
        "Assignment doesn't copy lists."),

    # ---- Dictionaries ----
    output("Dictionaries", "easy", "d = {'a': 1}\nprint(d['a'])", ["1"],
           "Square brackets look up a value by key."),
    fill("Dictionaries", "medium",
         "Which dict method returns a value or a default if the key is missing? (name only)",
         ["get", "get()"], "d.get(key, default) never raises KeyError.",
         "Three letters."),
    output("Dictionaries", "medium", "d = {'x': 1, 'y': 2}\nprint(list(d.keys()))",
           ["['x', 'y']"], "keys() gives the keys; list() converts the view to a list."),
    mcq("Dictionaries", "hard", "Which of these CANNOT be a dictionary key?",
        ["A tuple of ints", "A string", "A list", "An integer"], 2,
        "Keys must be hashable; lists are mutable and therefore unhashable."),

    # ---- Functions ----
    mcq("Functions", "easy", "Which keyword defines a function?",
        ["func", "def", "function", "lambda only"], 1,
        "Functions are defined with 'def'."),
    output("Functions", "medium", "def add(a, b=2):\n    return a + b\nprint(add(3))", ["5"],
           "b defaults to 2, so add(3) returns 3 + 2."),
    output("Functions", "medium", "def f():\n    pass\nprint(f())", ["none"],
           "A function without a return statement returns None.",
           "Think about what 'pass' does."),
    fill("Functions", "hard",
         "Fill in the blank: an anonymous one-line function is created with the ______ keyword.",
         ["lambda"], "lambda x: x + 1 creates a small anonymous function."),
    output("Functions", "hard", "def f(x, acc=[]):\n    acc.append(x)\n    return acc\nf(1)\nprint(f(2))",
           ["[1, 2]"],
           "The default list is created ONCE, so it's shared between calls "
           "(the classic mutable-default-argument trap).",
           "Is the default list recreated on every call?"),

    # ---- OOP ----
    mcq("OOP", "easy", "What is the name of the constructor method in a Python class?",
        ["__init__", "__new_object__", "constructor", "__start__"], 0,
        "__init__ initialises a new instance."),
    fill("OOP", "easy", "The first parameter of an instance method is conventionally named ______ .",
         ["self"], "self refers to the current instance."),
    mcq("OOP", "medium", "How do you make class B inherit from class A?",
        ["class B(A):", "class B extends A:", "class B -> A:", "class B: A"], 0,
        "Put the parent class in parentheses."),
    output("OOP", "hard",
           "class A:\n    def hi(self):\n        return 'A'\nclass B(A):\n    def hi(self):\n"
           "        return 'B' + super().hi()\nprint(B().hi())", ["ba"],
           "B.hi() calls A.hi() through super(), so the result is 'B' + 'A'."),
]

TOPICS = sorted({q.topic for q in QUESTIONS})
LEVELS = ["easy", "medium", "hard"]


# --------------------------------------------------------------------------
# Progress storage
# --------------------------------------------------------------------------
def load_progress() -> dict:
    if PROGRESS_FILE.exists():
        try:
            return json.loads(PROGRESS_FILE.read_text())
        except (json.JSONDecodeError, OSError):
            pass
    return {"sessions": [], "topics": {}}


def save_progress(data: dict) -> None:
    try:
        PROGRESS_FILE.write_text(json.dumps(data, indent=2))
    except OSError as exc:
        print(f"(Could not save progress: {exc})")


# --------------------------------------------------------------------------
# Helpers
# --------------------------------------------------------------------------
def ask_choice(prompt: str, valid: List[str]) -> str:
    while True:
        value = input(prompt).strip().lower()
        if value in valid:
            return value
        print(f"  Please enter one of: {', '.join(valid)}")


def ask_int(prompt: str, low: int, high: int) -> int:
    while True:
        raw = input(prompt).strip()
        if raw.isdigit() and low <= int(raw) <= high:
            return int(raw)
        print(f"  Enter a number between {low} and {high}.")


def read_answer(question: Question) -> str:
    """Read (possibly multi-line) answer. Blank line ends multi-line output answers."""
    if question.kind == "output" and "\n" in question.answer[0]:
        print("Your answer (multiple lines allowed, blank line to finish):")
        lines = []
        while True:
            line = input("> ")
            if not line.strip():
                break
            lines.append(line.strip())
        return "\n".join(lines)
    return input("Your answer: ").strip()


def is_correct(question: Question, given: str) -> bool:
    given = given.strip().lower()
    if question.kind == "output":
        # Compare ignoring quote style and spacing differences
        norm = lambda s: s.replace('"', "'").replace(" ", "")
        return norm(given) in {norm(a) for a in question.answer}
    return given in question.answer


def grade(percent: float) -> str:
    if percent >= 90:
        return "A - Excellent!"
    if percent >= 75:
        return "B - Great job!"
    if percent >= 60:
        return "C - Good, keep practising."
    if percent >= 40:
        return "D - Review the topic and retry."
    return "F - Don't give up, try the easy level first."


# --------------------------------------------------------------------------
# Quiz engine
# --------------------------------------------------------------------------
def select_questions() -> Optional[List[Question]]:
    print("\nChoose a topic:")
    for i, t in enumerate(TOPICS, 1):
        print(f"  {i}. {t}")
    print(f"  {len(TOPICS) + 1}. Mixed (all topics)")
    pick = ask_int("Topic number: ", 1, len(TOPICS) + 1)
    pool = QUESTIONS if pick == len(TOPICS) + 1 else [
        q for q in QUESTIONS if q.topic == TOPICS[pick - 1]]

    print("\nChoose a difficulty:")
    for i, lvl in enumerate(LEVELS, 1):
        print(f"  {i}. {lvl.capitalize()}")
    print(f"  {len(LEVELS) + 1}. All levels")
    lv = ask_int("Level number: ", 1, len(LEVELS) + 1)
    if lv <= len(LEVELS):
        pool = [q for q in pool if q.level == LEVELS[lv - 1]]

    if not pool:
        print("No questions match that selection. Try another combination.")
        return None

    n = ask_int(f"How many questions (1-{len(pool)})? ", 1, len(pool))
    return random.sample(pool, n)


def run_quiz(questions: List[Question]) -> dict:
    score = correct = streak = 0
    max_score = len(questions) * POINTS_PER_QUESTION
    per_topic = {}
    start = time.time()

    for number, q in enumerate(questions, 1):
        print("\n" + "=" * 60)
        print(f"Question {number}/{len(questions)}   [{q.topic} | {q.level}]")
        print("=" * 60)
        print(q.text)
        if q.kind == "mcq":
            for i, opt in enumerate(q.options):
                print(f"  {chr(65 + i)}. {opt}")
        if q.hint:
            print("(type 'hint' for a hint, costs "
                  f"{HINT_PENALTY} points)")

        hint_used = False
        while True:
            given = read_answer(q)
            if given.lower() == "hint" and q.hint:
                print(f"  Hint: {q.hint}")
                hint_used = True
                continue
            if given:
                break
            print("  Please type an answer.")

        stats = per_topic.setdefault(q.topic, {"asked": 0, "correct": 0})
        stats["asked"] += 1

        if is_correct(q, given):
            gained = POINTS_PER_QUESTION - (HINT_PENALTY if hint_used else 0)
            if streak >= 1:
                gained += STREAK_BONUS * streak
            streak += 1
            correct += 1
            score += gained
            stats["correct"] += 1
            print(f"Correct! +{gained} points" +
                  (f"  (streak x{streak})" if streak > 1 else ""))
        else:
            streak = 0
            shown = q.answer[0].upper() if q.kind == "mcq" else q.answer[0]
            print(f"Not quite. Correct answer: {shown}")
        print(f"Explanation: {q.explanation}")

    elapsed = int(time.time() - start)
    percent = 100 * correct / len(questions)
    print("\n" + "#" * 60)
    print("QUIZ COMPLETE")
    print(f"Correct: {correct}/{len(questions)} ({percent:.0f}%)")
    print(f"Score:   {score} points (base max {max_score})")
    print(f"Time:    {elapsed // 60}m {elapsed % 60}s")
    print(f"Grade:   {grade(percent)}")
    print("#" * 60)

    return {"date": time.strftime("%Y-%m-%d %H:%M"), "asked": len(questions),
            "correct": correct, "score": score, "per_topic": per_topic}


def update_progress(progress: dict, result: dict) -> None:
    progress["sessions"].append({k: result[k] for k in ("date", "asked", "correct", "score")})
    for topic, s in result["per_topic"].items():
        t = progress["topics"].setdefault(topic, {"asked": 0, "correct": 0})
        t["asked"] += s["asked"]
        t["correct"] += s["correct"]
    save_progress(progress)


def show_progress(progress: dict) -> None:
    print("\n--- Your progress ---")
    if not progress["sessions"]:
        print("No quizzes taken yet. Start one from the main menu!")
        return
    print(f"Quizzes taken: {len(progress['sessions'])}")
    print(f"Best score:    {max(s['score'] for s in progress['sessions'])}")
    print("\nAccuracy by topic:")
    for topic in TOPICS:
        t = progress["topics"].get(topic)
        if not t:
            print(f"  {topic:<14} not attempted")
            continue
        pct = 100 * t["correct"] / t["asked"]
        bar = "#" * int(pct // 10) + "." * (10 - int(pct // 10))
        print(f"  {topic:<14} [{bar}] {pct:5.1f}%  ({t['correct']}/{t['asked']})")
    weakest = min(progress["topics"].items(),
                  key=lambda kv: kv[1]["correct"] / kv[1]["asked"], default=None)
    if weakest:
        print(f"\nSuggestion: practise '{weakest[0]}' next.")
    print("\nRecent sessions:")
    for s in progress["sessions"][-5:]:
        print(f"  {s['date']}  {s['correct']}/{s['asked']} correct, {s['score']} pts")


# --------------------------------------------------------------------------
# Main menu
# --------------------------------------------------------------------------
def main() -> None:
    progress = load_progress()
    print("=" * 60)
    print("        PYTHON QUIZ LEARNING APP")
    print("=" * 60)

    while True:
        print("\nMain menu")
        print("  1. Start a quiz")
        print("  2. View progress")
        print("  3. Reset progress")
        print("  4. Quit")
        choice = ask_choice("Choose 1-4: ", ["1", "2", "3", "4"])

        if choice == "1":
            questions = select_questions()
            if questions:
                result = run_quiz(questions)
                update_progress(progress, result)
        elif choice == "2":
            show_progress(progress)
        elif choice == "3":
            if ask_choice("Really erase all progress? (y/n): ", ["y", "n"]) == "y":
                progress = {"sessions": [], "topics": {}}
                save_progress(progress)
                print("Progress reset.")
        else:
            print("Happy coding! Goodbye.")
            break


if __name__ == "__main__":
    try:
        main()
    except (KeyboardInterrupt, EOFError):
        print("\nGoodbye!")
