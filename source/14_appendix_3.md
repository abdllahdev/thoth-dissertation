# Appendix 3: Applications Source Code

**Todo App**

\begin{lstlisting}[caption=Todo app source code.,label={lst:todo-app-source-code},language=Thoth]
app Todo {
	title: "Todo App Created by Ra",
	auth: {
		userModel: User,
		idField: id,
		isOnlineField: isOnline,
		lastActiveField: lastActive,
		usernameField: username,
		passwordField: password,
		onSuccessRedirectTo: "/",
		onFailRedirectTo: "/login"
	}
}

model User {
	id         Int       @id
	username   String    @unique
	password   String
	isOnline	 Boolean	 @default(false)
	lastActive DateTime  @default(Now)
	tasks 		 Task[]
}

model Task {
	id        Int      @id
	title     String
	isDone    Boolean  @default(false)
	user 			User 		 @relation(userId, id)
	userId		Int
	createdAt DateTime @default(Now)
	updatedAt DateTime @updatedAt
}

@model(Task)
@permissions(IsAuth, OwnsRecord)
query<FindMany> getTasks {
	search: [ title, isDone ]
}

@model(Task)
@permissions(IsAuth)
query<Create> createTask {
	data: {
		fields: [title],
		relationFields: {
			user: connect id with userId
		}
	}
}

@model(Task)
@permissions(IsAuth, OwnsRecord)
query<Delete> deleteTaskById {
	where: id
}

component<Create> TaskCreateForm {
	actionQuery: createTask(),
	globalStyle: {
		formContainer: "flex mt-4"
	},
	formInputs: {
		title: {
			style: "flex w-full",
			input: {
				type: TextInput,
				placeholder: "Enter task title",
				isVisible: true,
				style: "shadow border rounded py-2 px-3 w-full mr-4 text-grey-darker"
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
		name: "Create",
		style: "rounded-md bg-teal-500 text-white px-4 py-2"
	}
}

component<Delete> TaskDeleteButton(id: Int) {
	actionQuery: deleteTaskById({
		where: id
	}),
	formButton: {
		name: "Delete",
		style: "rounded-md bg-red-500 text-white px-4 py-2"
	}
}

component TaskComponent(task: Task) {
	render(
		<div className="flex mb-4 items-center">
			<div className="w-full text-gray-500 text-xl">{ task.title }</div>
			<TaskDeleteButton id={task.id} />
		</div>
	)
}

component<FindMany> TasksComponent {
	findQuery: getTasks() as tasks,
	onError: render(<div>{ "An error occurred" }</div>),
	onLoading: render(<div>{ "Loading..." }</div>),
	onSuccess: render(
		<>
			[% for task in tasks %]
				<TaskComponent task={ task } />
			[% endfor %]
		</>
	)
}

component<SignupForm> MySignupForm {
	globalStyle: {
		formContainer: "bg-white shadow-md rounded px-8 pt-6 pb-8 mb-4",
		input: "appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline",
		inputLabel: "block text-gray-700 font-bold mb-2"
	},
	formInputs: {
		username: {
			label: {
				name: "Username"
			},
			input: {
				type: TextInput,
				placeholder: "Enter username"
			}
		},
		password: {
			label: {
				name: "Password"
			},
			input: {
				type: PasswordInput,
				placeholder: "Enter password"
			}
		}
	},
	formButton: {
		name: "Signup",
		style: "bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline"
	}
}

component<LoginForm> MyLoginForm {
	globalStyle: {
		formContainer: "bg-white shadow-md rounded px-8 pt-6 pb-8 mb-4",
		input: "appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline",
		inputLabel: "block text-gray-700 font-bold mb-2"
	},
	formInputs: {
		username: {
			label: {
				name: "Username"
			},
			input: {
				type: TextInput,
				placeholder: "Enter username"
			}
		},
		password: {
			label: {
				name: "Password"
			},
			input: {
				type: PasswordInput,
				placeholder: "Enter password"
			}
		}
	},
	formButton: {
		name: "Login",
		style: "w-full bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline"
	}
}

component<LogoutButton> MyLogoutButton {
	formButton: {
		name: "Logout",
		style: "inline-block align-baseline font-bold text-sm text-red-500 hover:text-red-800"
	}
}

@route("/")
@permissions(IsAuth)
page Tasks {
	render(
		<div className="flex h-screen items-center justify-center bg-slate-800">
			<div className="space-y-2">
				<div className="bg-white shadow-md rounded px-8 pt-6 pb-8 mb-4">
					<div className="mb-4">
						<h1 className="text-grey-darkest text-xl font-bold mb-4">{"Todo List"}</h1>
						<TaskCreateForm />
					</div>
					<TasksComponent />
				</div>
				<MyLogoutButton />
			</div>
		</div>
	)
}

@route("/login")
page LoginPage {
	render(
		<div className="flex h-screen items-center justify-center bg-slate-800">
			<div className="space-y-2">
				<span className="text-xl text-white">{"Login form"}</span>
				<MyLoginForm />
				<a className="inline-block align-baseline font-bold text-sm text-blue-500 hover:text-blue-800" href="/signup">
        	{ "Sign Up" }
     		</a>
			</div>
		</div>
	)
}

@route("/signup")
page SignupPage {
	render(
		<div className="flex h-screen items-center justify-center bg-slate-800">
			<div className="space-y-2">
				<span className="text-xl text-white">{"Sign up form"}</span>
				<MySignupForm />
				<a className="inline-block align-baseline font-bold text-sm text-blue-500 hover:text-blue-800" href="/signup">
        	{ "Login" }
     		</a>
			</div>
		</div>
	)
}
\end{lstlisting}

**Chatroom**

\begin{lstlisting}[caption=Chatroom source code.,label={lst:chatroom-source-code},language=Thoth]
app ChatRoom {
	title: "Chat Room Created by Ra",
	auth: {
		userModel: User,
		idField: id,
		isOnlineField: isOnline,
		lastActiveField: lastActive,
		usernameField: username,
		passwordField: password,
		onSuccessRedirectTo: "/",
		onFailRedirectTo: "/login"
	}
}

model User {
	id         Int       @id
	username   String    @unique
	password   String
	isOnline	 Boolean	 @default(false)
	lastActive DateTime  @default(Now)
	messages   Message[]
}

model Message {
	id        Int      @id
  content   String
	user 			User 		 @relation(userId, id)
	userId		Int
	createdAt DateTime @default(Now)
	updatedAt DateTime @updatedAt
}

@model(Message)
@permissions(IsAuth)
query<Create> createMessage {
	data: {
		fields: [content],
		relationFields: {
			user: connect id with userId
		}
	}
}

@model(Message)
@permissions(IsAuth)
query<FindMany> getMessages {
	search: [content]
}

@model(User)
@permissions(IsAuth)
query<FindMany> getUsers {
	search: [username]
}

component<FindMany> MessagesComponent {
	findQuery: getMessages() as messages,
	onError: render(
		<div>{ "An error occured" }</div>
	),
	onLoading: render(
		<div>{ "Loading..." }</div>
	),
	onSuccess: render(
    <div className="flex flex-col gap-2 flex-1">
			[% for message in messages %]
				<div
					key={message.id}
					className=[% if message.userId == LoggedInUser.id %]
											{ "flex gap-2 justify-end" }
										[% else %]
											{ "flex gap-2" }
										[% endif %]>
					<div
						className=[%  if message.userId == LoggedInUser.id %]
												{ "p-2 rounded-lg max-w-lg bg-blue-100" }
											[% else %]
												{ "p-2 rounded-lg max-w-lg bg-gray-100" }
											[% endif %]>
						<p>{message.content}</p>
						<span className="text-gray-500 text-sm">
							{message.user.username} {" • "}
							{message.createdAt}
						</span>
					</div>
				</div>
			[% endfor %]
    </div>
	)
}

component<FindMany> OnlineUsersComponent {
	findQuery: getUsers() as users,
	onError: render(
		<div>{ "An error occured" }</div>
	),
	onLoading: render(
		<div>{ "Loading..." }</div>
	),
	onSuccess: render(
		<div className="flex flex-col gap-2">
      <div className="font-bold text-lg mb-2">{ "Online Users" }</div>
			[% for user in users %]
				<div
          className=[% if user.id == LoggedInUser.id %]
											{ "flex items-center gap-2 font-bold" }
										[% else %]
											{ "flex items-center gap-2" }
										[% endif %]>
          <div
            className=[% if user.isOnline %]
												{ "w-3 h-3 rounded-full bg-green-500" }
											[% else %]
												{ "w-3 h-3 rounded-full bg-gray-500" }
											[% endif %]></div>
          <div>{ user.username }</div>
        </div>
			[% endfor %]
    </div>
	)
}

component<Create> CreateMessageForm  {
	globalStyle: {
		formContainer: "flex w-full"
	},
	actionQuery: createMessage(),
	formInputs: {
		content: {
			style: "flex w-full",
			input: {
				type: TextInput,
				placeholder: "Type your message here...",
				style: "flex-1 rounded-md border-gray-300 mr-2 px-4 py-2"
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
		name: "Send",
		style: "rounded-md bg-blue-500 text-white px-4 py-2"
	}
}

component<SignupForm> MySignupForm {
	globalStyle: {
		formContainer: "bg-white shadow-md rounded px-8 pt-6 pb-8 mb-4",
		input: "appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline",
		inputLabel: "block text-gray-700 font-bold mb-2"
	},
	formInputs: {
		username: {
			label: {
				name: "Username"
			},
			input: {
				type: TextInput,
				placeholder: "Enter username"
			}
		},
		password: {
			label: {
				name: "Password"
			},
			input: {
				type: PasswordInput,
				placeholder: "Enter password"
			}
		}
	},
	formButton: {
		name: "Signup",
		style: "bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline"
	}
}

component<LoginForm> MyLoginForm {
	globalStyle: {
		formContainer: "bg-white shadow-md rounded px-8 pt-6 pb-8 mb-4",
		input: "appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline",
		inputLabel: "block text-gray-700 font-bold mb-2"
	},
	formInputs: {
		username: {
			label: {
				name: "Username"
			},
			input: {
				type: TextInput,
				placeholder: "Enter username"
			}
		},
		password: {
			label: {
				name: "Password"
			},
			input: {
				type: PasswordInput,
				placeholder: "Enter password"
			}
		}
	},
	formButton: {
		name: "Login",
		style: "bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline"
	}
}

component<LogoutButton> MyLogoutButton {
	formButton: {
		name: "Logout",
		style: "w-full py-2 bg-red-500 text-white rounded-lg hover:bg-red-600"
	}
}

@route("/")
@permissions(IsAuth)
page Home {
	render(
    <div className="flex justify-center">
      <div className="flex flex-col w-full h-screen border shadow">
        <div className="flex flex-row h-full">
          <div className="bg-gray-100 flex flex-col w-1/4 p-4">
            <OnlineUsersComponent />
						<div className="flex justify-end mt-auto">
							<MyLogoutButton />
						</div>
          </div>
          <div className="flex flex-col w-3/4 p-4">
            <div className="flex-1 overflow-y-auto">
              <MessagesComponent />
            </div>
            <div className="bg-gray-100 flex flex-row p-4 rounded-lg">
              <div className="flex-1">
                <CreateMessageForm />
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
	)
}

@route("/login")
page LoginPage {
	render(
		<div className="flex h-screen items-center justify-center bg-slate-800">
			<div className="space-y-2">
				<span className="text-xl text-white">{"Login form"}</span>
				<MyLoginForm />
				<a className="inline-block align-baseline font-bold text-sm text-blue-500 hover:text-blue-800" href="/signup">
        	{ "Sign Up" }
     		</a>
			</div>
		</div>
	)
}

@route("/signup")
page SignupPage {
	render(
		<div className="flex h-screen items-center justify-center bg-slate-800">
			<div className="space-y-2">
				<span className="text-xl text-white">{"Sign up form"}</span>
				<MySignupForm />
				<a className="inline-block align-baseline font-bold text-sm text-blue-500 hover:text-blue-800" href="/signup">
        	{ "Login" }
     		</a>
			</div>
		</div>
	)
}
\end{lstlisting}

**Kanban Board**

\begin{lstlisting}[caption=Chatroom source code.,label={lst:kanban-board-source-code},language=Thoth]
app KanbanBoard {
	title: "Kanban Created by Ra",
	auth: {
		userModel: User,
		idField: id,
		isOnlineField: isOnline,
		lastActiveField: lastActive,
		usernameField: username,
		passwordField: password,
		onSuccessRedirectTo: "/",
		onFailRedirectTo: "/login"
	},
  clientDep: [
    ("react-beautiful-dnd", "^13.1.1"),
    ("@types/react-beautiful-dnd", "^13.1.3")
  ]
}

model User {
	id         Int       @id
	username   String    @unique
	password   String
	isOnline	 Boolean	 @default(false)
	lastActive DateTime  @default(Now)
	tasks      Task[]
}

model Task {
	id        Int      @id
	title     String
	user 			User 		 @relation(userId, id)
	userId		Int
  status    String   @default("todo")
	createdAt DateTime @default(Now)
	updatedAt DateTime @updatedAt
}

@model(Task)
@permissions(IsAuth)
query<FindMany> getTasks {
	search: [ status ]
}

@model(Task)
@permissions(IsAuth)
query<Create> createTask {
	data: {
		fields: [title],
		relationFields: {
			user: connect id with userId
		}
	}
}

@model(Task)
@permissions(IsAuth)
query<Update> updateTask {
	where: id,
	data: {
		fields: [status]
	}
}

@model(Task)
@permissions(IsAuth)
query<Delete> deleteTaskById {
	where: id
}

component<Delete> TaskDeleteButton(id: Int) {
	actionQuery: deleteTaskById({
		where: id
	}),
	formButton: {
		name: "Delete",
		style: "text-red-500"
	}
}

component<Create> TaskCreateForm {
	actionQuery: createTask(),
	formInputs: {
		title: {
			input: {
				type: TextInput,
				placeholder: "Enter task name",
				isVisible: true,
				style: "border border-gray-400 rounded px-3 py-2 w-80"
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
		name: "Create",
		style: "bg-green-500 hover:bg-green-600 text-white px-4 py-2 ml-2 rounded"
	}
}

component<Custom> TaskComponent(index: Int, task: Task) {
  imports: [|
    import { Draggable } from "react-beautiful-dnd";
		import TaskDeleteButton from "./TaskDeleteButton";
		import { Task } from "@/types";
  |],
  fn: [|
    return (
			<Draggable key={index} draggableId={task.id.toString()} index={index}>
				{(provided, snapshot) => (
					<div
						className={`border p-4 rounded-lg mb-2 space-y-2 ${
							snapshot.isDragging ? "bg-gray-100" : "bg-white"
						}`}
						ref={provided.innerRef}
						{...provided.draggableProps}
						{...provided.dragHandleProps}
					>
						<div className="flex justify-between items-center">
							<div className="font-bold">{task.title}</div>
							<TaskDeleteButton id={task.id} />
						</div>
						<div className="text-sm text-gray-500">
							Created at {new Date(task.createdAt).toLocaleString()} by{" "}
							{task.user.username}
						</div>
					</div>
				)}
			</Draggable>
    )
  |]
}

component<Custom> Column(columnId: String, columnName: String, tasks: Task[]) {
	imports: [|
		import { Droppable } from "react-beautiful-dnd";
		import TaskComponent from "./TaskComponent";
		import { Task } from "@/types";
	|],
	fn: [|
		if (!tasks) {
			return (
				<div >{ 'Loading...' }</div>
			)
		};

		return (
			<div
				className={`${columnId === "todo" && "bg-red-200"} ${
					columnId === "doing" && "bg-orange-200"
				} ${columnId === "done" && "bg-green-200"} w-90  rounded-lg p-4 mr-4`}
			>
				<h3 className="font-bold mb-4">{columnName}</h3>
				<Droppable droppableId={columnId}>
					{(provided) => (
						<div ref={provided.innerRef} {...provided.droppableProps}>
							{tasks &&
								tasks.map((task, index) => (
									<TaskComponent
										key={task.id}
										task={task}
										index={index}
									/>
								))}
							{provided.placeholder}
						</div>
					)}
				</Droppable>
			</div>
		)
	|]
}

component<Custom> KanbanBoard {
	imports: [|
		import { useState, useEffect } from "react";
		import { DragDropContext, DropResult } from "react-beautiful-dnd";
		import TaskCreateForm from "./TaskCreateForm";
		import Column from "./Column";
		import { User, Task } from "@/types";
	|],
	fn: [|
		type TaskStatus = "todo" | "doing" | "done";

		type Tasks = {
			todo: Task[];
			doing: Task[];
			done: Task[];
		};

		const storedUser = localStorage.getItem("LoggedInUser");
		const [LoggedInUser, _] = useState<User | undefined>(
			storedUser ? (JSON.parse(storedUser) as User) : undefined
		);

		const {
			data,
			isLoading,
			error,
		} = useFetch<Task[]>({
			findFunc: Queries.getTasks,
			eventsFunc: Queries.getTasksEvents,
			model: 'task',
			accessToken: LoggedInUser.accessToken,
		});

		const [tasks, setTasks] = useState<Tasks>({
			todo: [],
			doing: [],
			done: []
		});

		function categorizeTasks(tasks: Task[]): { [status in TaskStatus]: Task[] } {
			const categorizedTasks: { [status in TaskStatus]: Task[] } = {
				todo: [],
				doing: [],
				done: [],
			};

			if (tasks) {
				for (const task of tasks) {
					categorizedTasks[task.status as TaskStatus].push(task);
				}
			}

			return categorizedTasks;
		}

		useEffect(() => {
			if (data) {
				setTasks(categorizeTasks(data))
			}
		}, [data])

		const onDragEnd = async (result: DropResult) => {
			const { destination, source, draggableId } = result;

			if (!destination) {
				return;
			}

			if (destination.droppableId === source.droppableId && destination.index === source.index) {
				return;
			}

			const updateTask = Queries.updateTask({ where: draggableId })

			try {
				await fetch(updateTask, {
					method: 'put',
					headers: {
						'Content-Type': 'application/json',
						Authorization: `Bearer ${LoggedInUser?.accessToken}`,
					},
					body: JSON.stringify({ status: destination.droppableId }),
				});
			} catch (error) {
				console.log(error)
			}
		};

		return (
			<div className="flex flex-col items-center pt-10">
				<TaskCreateForm />
				<div className="flex justify-center pt-10">
					<DragDropContext onDragEnd={onDragEnd}>
						<div className="flex-grow">
							<Column
								columnId="todo"
								columnName="Todo"
								tasks={tasks.todo}
							/>
						</div>
						<div className="flex-grow">
							<Column
								columnId="doing"
								columnName="Doing"
								tasks={tasks.doing}
							/>
						</div>
						<div className="flex-grow">
							<Column
								columnId="done"
								columnName="Done"
								tasks={tasks.done}
							/>
						</div>
					</DragDropContext>
				</div>
			</div>
		)
	|]
}

component<SignupForm> MySignupForm {
	globalStyle: {
		formContainer: "bg-white shadow-md rounded px-8 pt-6 pb-8 mb-4",
		input: "appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline",
		inputLabel: "block text-gray-700 font-bold mb-2"
	},
	formInputs: {
		username: {
			label: {
				name: "Username"
			},
			input: {
				type: TextInput,
				placeholder: "Enter username"
			}
		},
		password: {
			label: {
				name: "Password"
			},
			input: {
				type: PasswordInput,
				placeholder: "Enter password"
			}
		}
	},
	formButton: {
		name: "Signup",
		style: "bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline"
	}
}

component<LoginForm> MyLoginForm {
	globalStyle: {
		formContainer: "bg-white shadow-md rounded px-8 pt-6 pb-8 mb-4",
		input: "appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline",
		inputLabel: "block text-gray-700 font-bold mb-2"
	},
	formInputs: {
		username: {
			label: {
				name: "Username"
			},
			input: {
				type: TextInput,
				placeholder: "Enter username"
			}
		},
		password: {
			label: {
				name: "Password"
			},
			input: {
				type: PasswordInput,
				placeholder: "Enter password"
			}
		}
	},
	formButton: {
		name: "Login",
		style: "bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline"
	}
}

component<LogoutButton> MyLogoutButton {
	formButton: {
		name: "Logout",
		style: "w-full py-2 bg-red-500 text-white rounded-lg hover:bg-red-600"
	}
}

@route("/")
@permissions(IsAuth)
page Home {
	render(
		<KanbanBoard />
	)
}

@route("/login")
page LoginPage {
	render(
		<div className="flex h-screen items-center justify-center bg-slate-800">
			<div className="space-y-2">
				<span className="text-xl text-white">{"Login form"}</span>
				<MyLoginForm />
				<a className="inline-block align-baseline font-bold text-sm text-blue-500 hover:text-blue-800" href="/signup">
        	{ "Sign Up" }
     		</a>
			</div>
		</div>
	)
}

@route("/signup")
page SignupPage {
	render(
		<div className="flex h-screen items-center justify-center bg-slate-800">
			<div className="space-y-2">
				<span className="text-xl text-white">{"Sign up form"}</span>
				<MySignupForm />
				<a className="inline-block align-baseline font-bold text-sm text-blue-500 hover:text-blue-800" href="/signup">
        	{ "Login" }
     		</a>
			</div>
		</div>
	)
}
\end{lstlisting}
