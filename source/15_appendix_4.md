# Appendix 4: User Experiment

**Questions**

**Introduction**

1- How many years of programming experience do you have?

- None
- Less than 1 year
- 1-3 years
- 3-5 years
- More than 5 years

2- What programming languages are you comfortable working with? (Select all that apply)

- Java
- JavaScript
- TypeScript
- Python
- Rust
- Ruby
- PHP
- None

3- How would you rate your web development skills?

- None
- Beginner
- Intermediate
- Advanced
- Expert

4- What web development technologies are you familiar with? (Select all that apply)

- HTML
- CSS
- JavaScript
- None

5- What frontend frameworks are you familiar with? (Select all that apply)

- React
- Qwik
- Svelte
- SolidJS
- Angular
- jQuery
- None

6- What backend frameworks are you familiar with? (Select all that apply)

- Django
- Flask
- Ruby on Rails
- Laravel
- NodeJS/ExpressJS
- Other

7- Have you used an ORM before?

- Yes
- No

8- Are you familiar with Prisma ORM?

- Yes
- No

**Syntax Comprehension Test**

9- How easy is it to understand this code snippet on a scale of 1 to 5, where a higher number indicates that the task is easier?

\begin{lstlisting}[caption=User experiment - Code snippet 1.,label={lst:code-snippet-1},language=Thoth]
model Task {
  id        Int      @id
  title     String
  isDone    Boolean  @default(false)
  createdAt DateTime @default(Now)
  updatedAt DateTime @updatedAt
}
\end{lstlisting}

10- In few words, please explain what did you understand (leave it empty if you did not understand anything)

11- How easy is it to understand this code snippet on a scale of 1 to 5, where a higher number indicates that the task is easier?

\begin{lstlisting}[caption=User experiment - Code snippet 2.,label={lst:code-snippet-2},language=Thoth]
model User {
  id         Int       @id
  tasks 		 Task[]
}

model Task {
  id        Int      @id
  title     String
  user 			User 		 @relation(userId, id)
  userId		Int
}
\end{lstlisting}

12- In few words, please explain what did you understand (leave it empty if you did not understand anything)

13- How easy is it to understand this code snippet on a scale of 1 to 5, where a higher number indicates that the task is easier?

\begin{lstlisting}[caption=User experiment - Code snippet 3.,label={lst:code-snippet-3},language=Thoth]
model Task {
  id         Int        @id
  title      String
  categories Category[]
}

model Category {
  id    Int    @id
  name  String
  tasks Task[]
}
\end{lstlisting}

14- In few words, please explain what did you understand (leave it empty if you did not understand anything)

15- How easy is it to understand this code snippet on a scale of 1 to 5, where a higher number indicates that the task is easier?

\begin{lstlisting}[caption=User experiment - Code snippet 4.,label={lst:code-snippet-4},language=Thoth]
@model(Task)
@permissions(IsAuth, OwnsRecord)
query<FindMany> getTasks {
  search: [ title, isDone ]
}

component TaskDetailComponent(task: Task) {
  render(
    <div className="flex mb-4 items-center">
      <div className="w-full text-gray-500 text-xl">{ task.title }</div>
    </div>
  )
}

component<FindMany> TaskListComponent {
  findQuery: getTasks() as tasks,
  onError: render(<div>{ "Error fetching tasks" }</div>),
  onLoading: render(<div>{ "Loading tasks..." }</div>),
  onSuccess: render(
    <>
      [% for task in tasks %]
        <TaskComponent task={ task } />
      [% endfor %]
    </>
  )
}
\end{lstlisting}

16- In few words, please explain what did you understand (leave it empty if you did not understand anything)

17- How easy is it to understand this code snippet on a scale of 1 to 5, where a higher number indicates that the task is easier?

\begin{lstlisting}[caption=User experiment - Code snippet 5.,label={lst:code-snippet-5},language=Thoth]
@model(Task)
@permissions(IsAuth)
query<Create> createTask {
  data: {
    fields: [ title ],
    relationFields: {
      user: connect id with userId
    }
  }
}

component<Create> TaskCreateForm {
  actionQuery: createTask(),
  formInputs: {
    title: {
      input: {
        type: TextInput,
        placeholder: "Enter task title"
      }
    },
    user: {
      input: {
        type: RelationInput,
        isVisible: false,
        defaultValue: connect id with LoggedInUser.id
      }
    }
  },
  formButton: {
    name: "Create"
  }
}
\end{lstlisting}

18- In few words, please explain what did you understand (leave it empty if you did not understand anything)

19- How easy is it to understand this code snippet on a scale of 1 to 5, where a higher number indicates that the task is easier?

\begin{lstlisting}[caption=User experiment - Code snippet 6.,label={lst:code-snippet-6},language=Thoth]
@model(Task)
@permissions(IsAuth, OwnsRecord)
query<Delete> deleteTaskById {
  where: id
}

component<Delete> TaskDeleteButton(id: Int) {
  actionQuery: deleteTaskById({
    where: id
  }),
  formButton: {
    name: "Delete"
  }
}
\end{lstlisting}


20- In few words, please explain what did you understand (leave it empty if you did not understand anything)

**Developer Experience Test**

21- In few words, can you explain why the compiler is raising this error and how it can be fixed? (Leave it empty if you don't know)

  Compiler error

  UndefinedError("@(Line:15): Undefined field 'name' in Model 'Task'")	"In few words, can you explain

\begin{lstlisting}[caption=User experiment - Code snippet 7.,label={lst:code-snippet-7},language=Thoth]
model Task {
	id        Int      @id
	title     String
	isDone    Boolean  @default(false)
	createdAt DateTime @default(Now)
	updatedAt DateTime @updatedAt
}

@model(Task)
@permissions(IsAuth)
query<Create> createTask {
  data: {
    fields: [ title, name ],
    relationFields: {
      user: connect id with userId
    }
  }
}
\end{lstlisting}

22- In few words, can you explain why the compiler is raising this error and how it can be fixed? (Leave it empty if you don't know)

  Compiler error

  TypeError("@(Line:20): Excepted a value of type 'Task' but received 'task' of type 'String' in 'TaskDetailComponent'")

\begin{lstlisting}[caption=User experiment - Code snippet 8.,label={lst:code-snippet-8},language=Thoth]
component TaskDetailComponent(task: Task) {
  render(
    <div className="flex mb-4 items-center">
      <div className="w-full text-gray-500 text-xl">{ task.title }</div>
    </div>
  )
}

component<FindMany> TaskListComponent {
  findQuery: getTasks() as tasks,
  onError: render(<div>{ "Error fetching tasks" }</div>),
  onLoading: render(<div>{ "Loading tasks..." }</div>),
  onSuccess: render(
    <>
      [% for task in tasks %]
        <TaskComponent task={ "task" } />
      [% endfor %]
    </>
  )
}
\end{lstlisting}

23- In few words, can you explain why the compiler is raising this error and how it can be fixed? (Leave it empty if you don't know)

  Compiler error

  FormInputTypeError("@(Line:6): Expected an input of type 'TextInput' for field 'title' but an input of type 'NumberInput' was implemented instead in 'TaskCreateForm'")

\begin{lstlisting}[caption=User experiment - Code snippet 9.,label={lst:code-snippet-9},language=Thoth]
component<Create> TaskCreateForm {
  actionQuery: createTask(),
  formInputs: {
    title: {
      input: {
        type: NumberInput,
        placeholder: "Enter task title"
      }
    },
    user: {
      input: {
        type: RelationInput,
        isVisible: false,
        defaultValue: connect id with LoggedInUser.id
      }
    }
  },
  formButton: {
    name: "Create"
  }
}
\end{lstlisting}

24- In few words, can you explain why the compiler is raising this error and how it can be fixed? (Leave it empty if you don't know)

  Compiler error

  TypeError("@(Line:4): Excepted a value of type 'String' but received 'false' of type 'Boolean' in '@default'")

\begin{lstlisting}[caption=User experiment - Code snippet 10.,label={lst:code-snippet-10},language=Thoth]
model Task {
  id        Int      @id
  title     String
  isDone    String   @default(false)
  user 			User 		 @relation(userId, id)
  userId		Int
  createdAt DateTime @default(Now)
  updatedAt DateTime @updatedAt
}
\end{lstlisting}

25- In few words, can you explain why the compiler is raising this error and how it can be fixed? (Leave it empty if you don't know)

  Compiler error

  UniqueFieldError("@(Line:14): Excepted a field of type 'UniqueField' but received 'title' of type 'NonUniqueField' in 'deleteTaskById'")

\begin{lstlisting}[caption=User experiment - Code snippet 11.,label={lst:code-snippet-11},language=Thoth]
model Task {
  id        Int      @id
  title     String
  isDone    Boolean  @default(false)
  createdAt DateTime @default(Now)
  updatedAt DateTime @updatedAt
}

@model(Task)
@permissions(IsAuth, OwnsRecord)
query<Delete> deleteTaskById {
  where: title
}
\end{lstlisting}

**Reflection**

26- How do you feel about the syntax on a scale of 1 to 5, where a higher number indicates that the bigger the better?

27- Were there any particular aspects of the DSL syntax that you found challenging or confusing?

28- Were there any aspects of the DSL that you feel could be improved or made more user-friendly?

29- On a scale of 1 to 5, where a higher number indicates that the bigger the better, how helpful were the compiler messages in identifying and fixing bugs in the DSL code?

30- Which of the following compiler messages that you found particularly helpful or informative? (Select all that apply)

- UndefinedError
- TypeError
- UniqueFieldError
- FormInputTypeError

31- Would you use the DSL or similar tool for personal projects?

- Yes
- No
- Maybe

32- What size would this DSL be helpful for?

- Small projects
- Medium projects
- Large projects
- None

**Results**

\begin{figure}[H]
  \centering
  \begin{subfigure}{0.45\textwidth}
    \centering
    \includegraphics[width=\textwidth]{source/figures/programming-experience.pdf}
    \caption{}
    \label{fig:programming-experience}
  \end{subfigure}
  \hspace{1cm}
  \begin{subfigure}{0.45\textwidth}
    \centering
    \includegraphics[width=\textwidth]{source/figures/web-experience.pdf}
    \caption{}
    \label{fig:web-experience}
  \end{subfigure}
  \caption{Participants' programming and web experience.}
  \label{fig:experience}
\end{figure}

\begin{figure}[H]
  \centering
  \begin{subfigure}{0.30\textwidth}
    \centering
    \includegraphics[width=\textwidth]{source/figures/1.pdf}
    \caption{}
    \label{fig:1}
  \end{subfigure}
  \hspace{0.5cm}
  \begin{subfigure}{0.30\textwidth}
    \centering
    \includegraphics[width=\textwidth]{source/figures/2.pdf}
    \caption{}
    \label{fig:2}
  \end{subfigure}
  \hspace{0.5cm}
  \begin{subfigure}{0.30\textwidth}
    \centering
    \includegraphics[width=\textwidth]{source/figures/3.pdf}
    \caption{}
    \label{fig:3}
  \end{subfigure}
  \begin{subfigure}{0.30\textwidth}
    \centering
    \includegraphics[width=\textwidth]{source/figures/4.pdf}
    \caption{}
    \label{fig:1}
  \end{subfigure}
  \hspace{0.5cm}
  \begin{subfigure}{0.30\textwidth}
    \centering
    \includegraphics[width=\textwidth]{source/figures/5.pdf}
    \caption{}
    \label{fig:2}
  \end{subfigure}
  \hspace{0.5cm}
  \begin{subfigure}{0.30\textwidth}
    \centering
    \includegraphics[width=\textwidth]{source/figures/6.pdf}
    \caption{}
    \label{fig:3}
  \end{subfigure}
  \caption{The rating distribution of each code snippet, as rated by the participants.}
  \label{fig:experience}
\end{figure}

\begin{longtable}{ABCD}
  \hline
  \textbf{Question No.} & \textbf{Participant A (25 min)} & \textbf{Participant B (60 min)} & \textbf{Participant C (30 min)} \\
  \hline
  \endfirsthead

  % Define the table footer
  \hline
  \multicolumn{4}{r}{Continued on next page...} \\
  \endfoot

  % Define the table last footer
  \hline
  \endlastfoot

  % Define the table content

  1 & More than 5 Years & More than 5 years & More than 5 years \\

  \hline

  2

  & Python

  & Java, JavaScript, Python, Rust

  & Java, JavaScript, Python, TypeScript \\

  \hline

  3 & None & Beginner & Beginner \\

  \hline

  4 & HTML, CSS, JavaScript & HTML, CSS, JavaScript & HTML, CSS, JavaScript \\

  \hline

  5 & None & jQuery & React \\

  \hline

  6 & NodeJS/ExpressJS & Django, Flask & Django, NodeJS/ExpressJS \\

  \hline

  7 & No & No & No \\

  \hline

  8 & No & No & No \\

  \hline

  9 & 5 & 4 & 4 \\

  \hline

  10

  & an entity called Task that has properties (unique id, title, is done? property, the time it is updated and created)

  & It defines the data model for a task which is described by five fields. The first column is the field's name, the second column is its data type, and the third is for extra attributes. @default specifies a field's default value. @id specifies that a field is a unique identifier/primary key. I'm not sure what @updatedAt does.

  & A Task model is being defined with 5 attributes (eg. id). Each attribute has an expected datatype. For example, the value of isDone is expecated to be a boolean, true or false. Some of the attributes are initialised with a default value if one isn't provided at the time of instantiating this model. For example, the value of isDone will be set to false if a value isn't provided. \\

  \hline

  11 & 5 & 5 & 3 \\

  \hline

  12

  & Same as the previous question but has user assigned to it

  & This defines two entities User and Task. A User has an integer ID which is its primary key and an array of tasks. A task has an integer ID, a string title, a user ID and a reference to the User with that ID. @relation describes a relation using a foreign key and the primary key it relates to. In other words there is a one-to-many relation from User to Task.

  & Two model/object definitions. One is of type User, one is of type Task. The User model has an attribute/property called tasks and it is expected to be an array of elements that are of type Task. The Task model has a property called user which is of type User and declares a relationship between its own userId property and the User model's id property. \\

  \hline

  13 & 5 & 5 & 4 \\

  \hline

  14

  & Same as the previous but has a category and the category has its own properties

  & This describes two entities Task and Category which have a many-to-many relationship. Both entities have an integer ID and a string title/name. A Task has an array of categories and a Category has an array of tasks.

  & Two models/objects being defined with a set of properties and those properties expected datatypes. The Category object accepts an array of Task objects for the tasks property and the Task object accepts an array of Category objects for the categories property. \\

  \hline

  15 & 2 & 3 & 5 \\

  \hline

  16

  & It is getting all the tasks, then visualizing the tasks

  & getTasks is a query that searches for many Tasks according to the tasks' 'title' and 'isDone' fields. It has the IsAuth and OwnsRecord permissions.

  TaskDetailComponent is a UI element that takes a task as a parameter, and displays a its title inside some formatted <div>s.

  TasksListComponent is a UI element that uses the getTasks query to obtain a list of tasks. While the query is loading a message is displayed. If the query succeeds each task is displayed in a TaskDetailComponent. If an error occurs an error message is displayed.

  & getTasks: A query to get tasks is defined to extract and store the title and the value of isDone for all the tasks available in the record. It can be executed on the Task model. The requirements for this to be allowed involve the end user's account being authenticated and owning a record.

  TaskDetailComponent: A definition of a component that takes in an object of type Task and consists of two nested div blocks. The outer one centres the inner div. The inner div allows the title of the task to be rendered in an extra large grey font.

  TasksListComponent: The returned values from the getTasks query are stored as a variable called tasks. If there is an error while executing the getTasks query, a div block is rendered on the page, informing the user that there was an "Error fetching tasks". While attempting to retrieve the tasks, the div block is rendered with the "Loading tasks" text. If the getTasks query is executed completely with no issues, a list of TaskDetailComponents are rendered on the page based on the output of the query where each TaskDetailComponent displays information about each task in the list. \\

  \hline

  17 & 1 & 3 & 5 \\

  \hline

  18

  & N/A

  & createTask is a query that creates a Task by providing its title and relating its userId to a user.

  TaskCreateForm is a UI component that is a form which triggers the createTask query when submitted. It has a text input field for the task's title and a hidden input for the user's ID, as well as a button to submit the form. The title input has some placeholder text. The user input uses the ID of the logged-in user.

  & The end user must have an authenticated account for the following code to be executed. It allows the end user to fill in a form where they can create a task \\

  \hline

  19 & 5 & 4 & 5 \\

  \hline

  20

  & Implemeting a delete tas functionality in UI with query

  & deleteTaskById is a query for deleting the Task that has a particular ID.

  TaskDeleteButton is a UI element that is parameterised by an integer ID. It defines a button with the text "Delete" which triggers the deleteTaskById query using the given ID.

  & User must be logged in and have an existing record. The code allows the user to delete a task through the use of a delete button \\

  \hline

  21

  & there is no name property in Task Model

  & The createTask query specifies a 'name' field for a Task, but Tasks do not have a field called 'name'. You could remove 'name' from line 15 so that it only contains 'title', or add a field called 'name' to Task.

  & The name field does not exist in the Task model. This needs to be defined in the model with a datatype (eg String). \\

  \hline

  22

  & it should not be inside "" it should be only the variable task

  & Instead of providing the 'task' variable to each TaskDetailComponent, the string literal "task" is used. Remove the double quotes in line 20.

  & The TaskDetailComponent component expects a Task object being passed in. Line 20 passes in a string. The value, task, can be passed in without speech marks to solve the problem. \\

  \hline

  23

  & The title should have TextInput instead of the NumberInput

  & The 'title' input field is of type NumberInput but should be a TextInput. Replace NumberInput in line 6 with TextInput.

  & The title field should be a string and therefore the user input should be of type TextInput rather than NumberInput as this would mean the title is of the wrong type for the model \\

  \hline

  24

  & As its default is boolean not string as it is defined type

  & The boolean value 'false' was provided as a default value for the String field 'isDone'. Replace 'String' with 'Boolean' in line 4.

  & The default value and expected datatype for the isDone field are contradicting each other. The datatype should be changed to Boolean to solve the problem. \\

  \hline

  25

  & as title is not unique it should have been id instead

  & The query deleteTaskById is deleting according to a Task's 'title' field, which is not a unique field. Replace 'title' in line 14 with 'id'.

  & The id should be used rather than the title as this is a unique identifier \\

  \hline

  26 & 4 & 4 & 4 \\

  \hline

  27

  & not really, maybe the part of ui was more challenging

  & It was not clear what some of the attributes (@something) do.

  The angle-bracket notation for different types of query was not clear at first. It resembles type parameters in C++ or Rust but I'm unsure whether 'Create', 'FindMany' and 'Delete' are types.

  & I only realised at the last question what the significance of the @ symbol was when defining the model. \\

  \hline

  28

  & maybe the closeness of the language to the human language

  & The language itself seems like it would be very usable after taking some time to learn it. Because it uses type inference, an editor plugin that enables inspecting the inferred types would be very helpful.

  & The syntax in general is really good and I could understand most of it. No improvements needed \\

  \hline

  29 & 5 & 5 & 5 \\

  \hline

  30

  & UndefinedError, TypeError, FormInputTypeError, UniqueFieldError

  & UndefinedError, TypeError, FormInputTypeError

  & UndefinedError, TypeError, FormInputTypeError, UniqueFieldError \\

  \hline

  31 & Yes & Yes & Maybe \\

  \hline

  32 & Medium projects & Medium projects & Medium projects \\

\end{longtable}

\newpage

\begin{longtable}{ABCD}
  \hline
  \textbf{Question No.} & \textbf{Participant D (31 min)} & \textbf{Participant E (46 min)} & \textbf{Participant F (22 min)} \\
  \hline
  \endfirsthead

  % Define the table footer
  \hline
  \multicolumn{4}{r}{Continued on next page...} \\
  \endfoot

  % Define the table last footer
  \hline
  \endlastfoot

  % Define the table content

  1

  & More than 5 Years

  & More than 5 Years

  & More than 5 years \\

  \hline

  2

  & Java, JavaScript, Python

  & Java, JavaScript, TypeScript, Python

  & TypeScript, JavaScript, PHP, Python \\

  \hline

  3

  & Intermediate

  & Advanced

  & Advanced \\

  \hline

  4

  & HTML, CSS, JavaScript

  & HTML, CSS, JavaScript

  & HTML, CSS, JavaScript \\

  \hline

  5

  & React, Angular, jQuery

  & React, Angular, jQuery

  & React, Angular, jQuery \\

  \hline

  6

  & Flask, NodeJS/ExpressJS

  & Django, Flask, NodeJS/ExpressJS

  & Laravel, NodeJS/ExpressJS, Flask \\

  \hline

  7 & No & Yes & Yes \\

  \hline

  8 & No & Yes & Yes \\

  \hline

  9 & 5 & 4 & 5 \\

  \hline

  10

  & it defines a model called Task which has 5 properties and of designed types. Id is of type Int and is unique. Isdone and createat have respectively default values when created and not assigned.

  & This is a data model representing a Task. The Task has the field id which is an integer and is also the Task's unique identifier. The Task also has the fields title, which is a string, isDone which is a boolean defaulted to false (so when no value is provided, it will be false), createdAt which is a datetime defaulted to the time of the Task object's instantiation, and updatedAt which is a datetime that is automatically updated every time the Task object is updated.

  & I understand that the snippets above defines a database model, where `id` field is an integer primary key, `title` is a string field, `isDone` is a boolean field with a default value of false, `createdAt` is a timestamp field with default value as the current server's timestamp, I guessed that`@updatedAt` notation maps the field to the time of the most recent update. \\

  \hline

  11 & 4 & 5 & 5 \\

  \hline

  12

  & 2 models. Similar to previous one. I don’t understand what the parameters for @relation are, weather the id for the parameters refers to task or user. Also why wr need userid and user entity belongs to task together. The info for user and userid should be defined separately if considering database / software design?

  & The code snippet shows User and Task models. The User has an id field which is an integer, and is the User model's unique identifier. User also has a tasks field which is an array of the Task objects. The Task model has an id field which is an integer, and is the Task model's unique identifier. Task also has a title field which is a string, user field which is actually a relation representing a User object, and the relation's id (userid, which is a foreign key). This establishes one-to-many relationship, where one User can have many Tasks, but one Task can only be tied to one User.

  & `User` model has a one to many relationship with `Task` model, the foreign key on `Task` is `userId` field. \\

  \hline

  13 & 1 & 5 & 4 \\

  \hline

  14

  & There’s a definition loop

  & The code snippet shows Task and Category models. The Task model has an id field which is an integer, and is the Task model's unique identifier. Task also has a title field which is a string, and a categories field which is an array of Category objects. The Category model has an id field which is an integer, and is the Category model's unique identifier. Category also has a name field which is a string, and a tasks field which is an array of Task objects. I can interpret from this code snippet that this establishes a many-to-many relationship between Task and Category, where one Task can have many Categories, and one Category can have many Tasks.

  & A many-to-many relationship. It's a bit confusing, but I believe it implies that the schema parser would automatically create pivot/join table and connect records? \\

  \hline

  15 & 5 & 5 & 4 \\

  \hline

  16

  & Reads tasks as list and use each data to render a component for all data. List rendering deals with error etc cases.

  & TaskCreateForm is a component that neatly represents a form. This form has title and user inputs. This form is used to create a task. The input type for the title is TextInput, meaning just text. The input type for the user is RelationInput. The user input is not visible, as the user does not manually enter it, instead it is automatically filled using the logged in user's ID. When the form is submitted, by clicking the button that says "Create", it invokes the createTask() query, specified by actionQuery in the form component. The createTask query then uses the inputs to the form to create a Task record. The user that clicks 'create' can only create a Task record using this form if they are authenticated.

  & Seems like a field relevant to Task database Model. I assume the permission annotation would only make it accessible for this who created the record.

  FindMany query definition at the top is a bit confusing, I am not sure if search defines searchable items or selected fields.

  TaskListComponent seems to be a data fetching handler and view manager for TaskList, I expect that findQuery is the data source, onError, onLoading, onSuccess is for handling views in different data fetching states. \\

  \hline

  17 & 4 & 5 & 5 \\

  \hline

  18

  & Renders a form but I don’t know what’s the connect syntax’s meaning

  & TaskCreateForm is a component that neatly represents a form. This form has title and user inputs. This form is used to create a task. The input type for the title is TextInput, meaning just text. The input type for the user is RelationInput. The user input is not visible, as the user does not manually enter it, instead it is automatically filled using the logged in user's ID. When the form is submitted, by clicking the button that says "Create", it invokes the createTask() query, specified by actionQuery in the form component. The createTask query then uses the inputs to the form to create a Task record. The user that clicks 'create' can only create a Task record using this form if they are authenticated.

  & Same as the previous answer, on model and permission fields. The create query creates a task and links it to the authenticated user. `TaskCreateForm` is defining the query that's to be executed on form submission, and the inputs to show in the frontend form. \\

  \hline

  19 & 5 & 5 & 5 \\

  \hline

  20

  & Renders a button and defines the delete JH behaviour for button

  & TaskDeleteButton is a component that simply represents a button. This button reads "Delete", and when clicked it invokes the deleteTaskById() query, specified by actionQuery inside the component. The deleteTaskById query takes a single 'where' argument, which specifies the ID for the Task record to delete. The user that clicks 'delete' can only delete the Task record if they are authorised and own the Task record that they are trying to delete.

  & Pretty much the same as the pervious one, but instead of Creation, it handles deletion form and query. \\

  \hline

  21

  & name probably should be replaced by title

  & The error arises because there is no 'name' field in the Task model. This can be fixed by removing 'name' from the fields in the createTask query, or by creating a 'name' field in the Task model.

  & The createTask query has a field that doesn't exist inside the `Task` model, can be fixed by either adding a `name` field to `Task` model or removing `name` from data fields. \\

  \hline

  22

  & Probably should remove “” for task

  & The error arises because you are passing the task prop to the TaskDetailComponent as a string. It is specified in the signature of TaskDetailComponent that the task prop should be of type Task, and not of type String. To fix this, remove the quotes around "task" on line 20.

  & The value passed to TaskDetailComponent doesn't match the expected type, can be fixed by replacing "task" with task. \\

  \hline

  23

  & Change numberinput to TextInput

  & The error arises because you have put the input type for the 'title' field in the form to be a NumberInput. To fix this, replace NumberInput with TextInput, because the title field in the Task model is a String not a Number/Integer.

  & The form input type is set to NumberInput, the `Task` database model maps this field to `String`. This can be fixed by using `StringInput` (if available idk) on line 6 \\

  \hline

  24

  & Change the type to Boolean or false to some string value

  & This error arises because you are passing a boolean default value (false) to a String field. To fix this, isDone should be of type Boolean not of type String. Alternatively, if you want isDone to be of type String then the default value should also be a String, e.g. "" (double empty quotes).

  & The default value for a string field is set to a boolean value. This can be fixed by replacing `String` value with `Boolean` on line 4 \\

  \hline

  25

  & Change title to id

  & This error arises because you are using 'title' as the where clause for selecting a task to delete in the deleteTaskById query. The title field is not a unique identifier for the Task model, and as such two Task records could have the same title field value, causing uncertainty when deleting on 'where title'. To fix this, you should replace 'title' on line 14 with 'id'. This is because id is the unique identifier for the Task model, highlighted by the @id attribute on line 2.

  & The delete query is depending on a non-unique field, which means that it can delete multiple records by mistake. Can be fixed by replacing title with id on line 14 \\

  \hline

  26 & 4 & 5 & 4 \\

  \hline

  27

  & N/A

  & The <> syntax was a little confusing, but this is because I am not so confident with generics in general. I am assuming these are similar to generics, because of the <> syntax which represents generics in TypeScript. However, it did not impede my ability to understand the code syntax. In fact, in many cases it helped.

  & With the delete query it was a bit confusing how the DSL differs from DELETE and DELETE many. \\

  \hline

  28

  & Can’t think of

  & It would be good to have some documentation that describes the meaning of the values in the <> syntax, such as the different query types. Other than that, this syntax appear very easy to use and understand.

  & N/A \\

  \hline

  29 & 5 & 5 & 5 \\

  \hline

  30

  & UndefinedError, TypeError, FormInputTypeError, UniqueFieldError

  & UndefinedError, TypeError, FormInputTypeError, UniqueFieldError

  & UndefinedError, TypeError, FormInputTypeError, UniqueFieldError \\

  \hline

  31

  & Maybe

  & Yes

  & Maybe \\

  \hline

  32

  & Medium projects

  & Large projects

  & Small projects \\

\end{longtable}

\newpage

\begin{longtable}{ABCD}
  \hline
  \textbf{Question No.} & \textbf{Participant G (34 min)} & \textbf{Participant H (64 min)} & \textbf{Participant I (50 min)} \\
  \hline
  \endfirsthead

  % Define the table footer
  \hline
  \multicolumn{4}{r}{Continued on next page...} \\
  \endfoot

  % Define the table last footer
  \hline
  \endlastfoot

  % Define the table content

  1

  & 1-3 years

  & 1-3 years

  & 3-5 years \\

  \hline

  2

  & Java, Python

  & JavaScript, TypeScript, Python, PHP

  & Python, Rust \\

  \hline

  3

  & Beginner

  & Intermediate

  & Beginner \\

  \hline

  4

  & HTML, CSS

  & HTML, CSS, JavaScript

  & None \\

  \hline

  5

  & None

  & React, jQuery

  & None \\

  \hline

  6

  & Django

  & Laravel, NodeJS/ExpressJS

  & Django \\

  \hline

  7 & No & Yes & Yes  \\

  \hline

  8 & No & No & No \\

  \hline

  9 & 4 & 5 & 5 \\

  \hline

  10

  & A Task has 5 attributes (id, title, isDone, createdAt and updatedAt) each attribute is a different type, e.g id in an integer, title is string etc.

  & It is table for tasks that indicates if a task is done or not. Also, it specify its title and when it is added and updated.

  & This code defines a table within a database for storing tasks. Each task has a title, whether it is completed (false by default), when it was created (now by default) and when it was updated (automatically incremented on each update). The 'id' column is the primary key for each entry in this table \\

  \hline

  11 & 4 & 5 & 3 \\

  \hline

  12

  & Each user has an id and an array of tasks, each task has an id, title and a user that its connected to

  & one record in a table User can be associated with one or more records in table Task

  & There is a User table and a Task table within the database. Each task has a user relation, and a user can have multiple tasks related to them. What I don't properly understand is why there is a 'userId' and a 'user' in the Task model \\

  \hline

  13 & 3 & 5 & 4 \\

  \hline

  14

  & A task can have an id, title and be a part of many categories. A category has an id, a name and an array of tasks which are part of it

  & one or multiple records in a table Task can be associated with one or multiple records in table Categories.

  & This represents a many-to-many relationship between tasks and categories. \\

  \hline

  15 & 3 & 5 & 4 \\

  \hline

  16

  & I think it is trying to fetch all the tasks in the database and rendering it onto the webpage if successful.

  & fetch the done tasks and list them by title

  & The TasksListComponent executes the getTasks query, returning the 'title' and 'isDone' for all tasks owned by the user visiting the component. The TasksListComponent also has separate HTML defined for loading a page and if there is any error. On success of the query, the TasksListComponent renders the TaskDetailComponent for each of the user's tasks. \\

  \hline

  17 & 3 & 4 & 3 \\

  \hline

  18

  & I think this is creating the page in which the user can create a task where they enter a task title which is connected to that user. then they press 'create' button to create it

  & create a new task and link it by the logged in user

  & The TaskCreateForm takes as input a task title, as well as the user (but this is grabbed automatically based on who is logged in). The createTask query is executed with the user and title, to create a task with a user. The task is related to the user based on 'connecting' ids but I don't properly get this \\

  \hline

  19 & 4 & 5 & 5 \\

  \hline

  20

  & Deleting the task with the corresponding id. User can press 'Delete' button to delete it

  & just delete a task

  & This defines a TaskDeleteButton component that takes as input an ID, and executes a query to delete the Task with that id, provided the user owns that record. This button is also labelled with 'Delete'. \\

  \hline

  21

  & 'name' is not a valid attribute name in the Task model

  & Task model is created in a wrong way

  & In the createTask query, there is a reference to a 'name' field in Task, but this field does not exist in the Task model. To fix this, 'name' should be replaced with 'user'. \\

  \hline

  22

  & Here, "task" is a string but it needs to be of type task. Get rid of the quotation marks

  & task is passed as a string not as a variable

  & The TaskDetailComponent has been given the string "task" when it should have been passed a task object. To fix this, remove the quotation marks from "task" on line 20. \\

  \hline

  23

  & The type in line 6 should be TextInput instead of NumberInput as the user is entering a title for the task

  & title should be string

  & The Task title should be text instead of a number. To fix this, replace 'NumberInput' with 'TextInput' on line 6. \\

  \hline

  24

  & In line 4, the isDone attribute should be of type Boolean instead of string

  & isDone should be Boolean

  & isDone is defined to be a String type, but has been given a boolean default value. To fix this, either set the default value to be a string, such as "false", or better, change isDone to be a Boolean column. \\

  \hline

  25

  & title is not if type 'UniqueField', should be id instead

  & title is not unique field it should be replaced by id for example

  & The deleteTaskById query requires a unique field from the Task model. 'title' is not unique. To fix this, provide a unique field on line 14. In this case, we should replace 'title' with the id field on line 14 \\

  \hline

  26 & 4 & 4 & 4 \\

  \hline

  27

  & wasn't too sure what the @ signs meant but Im not very good at programming so could just be me

  & NO

  & I found the definition of relations to sometimes be confusing. The many-to-many syntax with [] square brackets was great and very simple!! But I was confused by the foreignkey 'connect' syntax for ids, as well as when the Task model required both a 'user' AND a userId to be specified in its model definition, in order to be linked to a User.

  At first I was confused a bit by the scope of some variables. For instance, some of the components referred to things such as 'title' and 'task', when they were never provided as inputs to the component. Although I did eventually understand that these were defined globally in the file. \\

  \hline

  28

  & no

  & N/A

  & Perhaps the syntax for 'connecting' ids as mentioned before, or at least it would need clear documentation. \\

  \hline

  29 & 5 & 4 & 5 \\

  \hline

  30

  & UndefinedError, FormInputTypeError

  & UndefinedError, TypeError, FormInputTypeError, UniqueFieldError

  & UndefinedError, TypeError, FormInputTypeError, UniqueFieldError \\

  \hline

  31

  & Yes

  & Yes

  & Maybe \\

  \hline

  32

  & Small projects

  & Medium projects

  & Medium projects \\

\end{longtable}
