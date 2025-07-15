# crewAI
Example of an Operations Research development workflow with developer and tester agents using crew AI

# Build Your First AI Team: A Step-by-Step Guide to crewAI
Ever faced a task so complex it felt like you needed a whole team to tackle it? What if you could build that team with AI? Welcome to the world of crewAI, a powerful framework for orchestrating autonomous AI agents who can work together to solve complex problems.

In this guide, we'll go from zero to hero, building a sophisticated AI team right inside a Google Colab notebook. Our crew will consist of a Python Developer and a QA Tester. Their mission: to write, test, and debug a Python script that solves the classic Traveling Salesman Problem (TSP) using a professional optimization library.

Let's dive in!

## Getting Started: Setup and API Keys

First, we need to set up our workspace and install the necessary libraries. crewAI is the core framework, crewai_tools provides useful add-ons, and ortools is the powerful optimization library our crew will use.

Run this command in a Colab cell:

Python
```
!pip install crewai crewai_tools ortools
```
Next, you'll need an OpenAI API key. The most secure way to handle this in Colab is by using the built-in Secrets Manager.

Click the key icon (🔑) on the left sidebar of your notebook.

Click "Add a new secret".

For the Name, use OPENAI_API_KEY.

In the Value field, paste your actual OpenAI API key.

Make sure the "Notebook access" toggle is on.

Now, use this code to load your key into the environment:

Python
```
import os
from google.colab import userdata

os.environ["OPENAI_API_KEY"] = userdata.get('OPENAI_API_KEY')
```
## The Building Blocks: Tools, Agents, and Tasks

A crew is made of three core components:

Tools ⚙️: Skills you give your agents, like running code or searching the web.

Agents 🤖: The AI workers, each with a specific role and goal.

Tasks 📝: The assignments you give to your agents.

Creating a Custom Tool

Our tester agent needs the ability to execute code. We can grant this skill by creating a custom tool. BaseTool lets us define a new capability from scratch.

Python
```
from crewai.tools import BaseTool
import io
from contextlib import redirect_stdout, redirect_stderr

class CodeExecutionTool(BaseTool):
    name: str = "Code Execution Tool"
    description: str = "Executes Python code and returns the output or error. Input should be a string of Python code."

    def _run(self, code: str) -> str:
        output_buffer = io.StringIO()
        error_buffer = io.StringIO()
        try:
            with redirect_stdout(output_buffer), redirect_stderr(error_buffer):
                exec(code, {})
            
            output = output_buffer.getvalue().strip()
            error = error_buffer.getvalue().strip()

            if error:
                return f"Execution failed with error: {error}"
            elif output:
                return f"Execution successful with output: {output}"
            else:
                return "Execution successful with no output."
        except Exception as e:
            return f"An exception occurred during execution: {str(e)}"

code_executor = CodeExecutionTool()
```
Designing Your AI Agents

Now, let's define our two agents. Notice how their role, goal, and backstory give them a clear persona. We'll equip our tester with the code_executor tool we just built.

Python
```
from crewai import Agent

# Developer Agent 🧑‍💻
developer = Agent(
    role='Senior Python Developer',
    goal='Write clean, efficient, and error-free Python code using the Google OR-Tools library to solve the Traveling Salesman Problem.',
    backstory=(
        "You are an experienced Python developer with a knack for solving complex algorithmic problems. "
        "You receive task descriptions and turn them into functional Python scripts, "
        "diligently debugging and refactoring based on QA feedback."
    ),
    verbose=True,
    allow_delegation=False
)

# Tester Agent 🧪
tester = Agent(
    role='Software Quality Assurance Engineer',
    goal='Test the Python code provided by the developer to ensure it is error-free and provides the optimal solution.',
    backstory=(
        "You are a meticulous QA engineer. Your job is to take Python scripts, "
        "run them using your execution tool, and rigorously test them. If you find any issue, "
        "you report it back to the developer with the error message and suggestions for how to fix it."
    ),
    tools=[code_executor], # Assign the tool here
    verbose=True,
    allow_delegation=True # Allow the tester to send tasks back to the developer
)
```
Defining the Mission: Tasks

Finally, we create the tasks. We'll define the problem and the expected outcome. The context parameter in testing_task is crucial—it tells the tester to use the code produced by the developer in the coding_task.

Python
```
from crewai import Task

# Task description for the developer
algorithm_description = (
    "Create a Python function that solves the Traveling Salesman Problem (TSP) using the "
    "Google OR-Tools library. "
    "\n\nThe function should be named `solve_tsp_ortools` and take one argument:"
    "\n1. `distance_matrix`: A 2D list representing the distances between cities."
    "\n\nThe implementation must use `ortools.constraint_solver.pywrapcp` to create a routing model, "
    "find the optimal solution, and return a tuple containing:"
    "\n1. `path`: A list of city indices for the optimal tour, starting and ending at city 0."
    "\n2. `distance`: The total distance of the optimal tour."
    "\n\nInclude a test case that calls the function with a sample distance matrix and prints the resulting path and distance."
)

# The coding task
coding_task = Task(
    description=f"Write a Python script that implements the following: {algorithm_description}",
    expected_output="A single block of clean, executable Python code that defines the `solve_tsp_ortools` function and includes a test case.",
    agent=developer
)

# The testing task
testing_task = Task(
    description=(
        "You are responsible for testing the Python code for the Traveling Salesman Problem. "
        "Use your 'Code Execution Tool' to run the script."
        "\nThe test case in the script should find an optimal solution with a total distance of 80."
        "\nVerify that the script's output contains 'Total Distance: 80'. "
        "\nIf the code runs and the distance is correct, your final answer is the code itself with a confirmation. "
        "If there is any error or the distance is not 80, delegate back to the developer with a report."
    ),
    expected_output="The final, correct Python code with the confirmation 'Code is tested and works perfectly with an optimal distance of 80.'",
    agent=tester,
    context=[coding_task]
)
```
## Launching the Crew 🚀

With our tools, agents, and tasks ready, it's time to assemble the crew and kick off the mission! We'll use a sequential process, where tasks are executed one after another. crewAI's delegation logic will handle the back-and-forth between the agents if the tester finds a bug.
Python
```
from crewai import Crew, Process

# Assemble the crew
code_crew = Crew(
    agents=[developer, tester],
    tasks=[coding_task, testing_task],
    process=Process.sequential,
    verbose=2 # Use verbose=2 for detailed, step-by-step logging
)

# Kick off the work!
result = code_crew.kickoff()

# Print the final result
print("\n\n########################")
print("## Crew Work Complete!")
print("########################\n")
print("Final Result:")
print(result)
```
When you run this, you'll see the agents "thinking" and passing work between each other. The developer will write the initial code, and the tester will run it. If it fails, the tester will send it back with an error report until the code is perfect!

## Conclusion

You did it! You've successfully built and launched a multi-agent AI team to solve a complex programming task. From installing libraries and defining custom tools to designing specialized agents and launching a crew, you've seen how crewAI can automate sophisticated workflows.

The possibilities are endless. You could build crews for market research, content creation, financial analysis, and so much more. Now it's your turn to get creative—what will your crew build next?
