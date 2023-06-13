\chapter{Analysis and Specification}
\label{chap:analysis-and-specs}

As discussed in the previous chapter, there are several existing tools in the literature designed to support multitier programming for full-stack apps and to address the issue of boilerplate code. Some of the previously introduced multitier programming languages faced challenges that hindered their widespread adoption. For instance, languages such as Links and Hop lacked interoperability with existing general-purpose languages, which made them lack support for tools required for web app development while Opa allowed developers to access JavaScript libraries. Languages like Links were built using a syntax and programming paradigm unfamiliar to most developers, making them difficult to learn and adopt.

As shown in Figure @fig:github-stars, Opa was the most preferred language compared to the other two. Our approach aims to create a DSL that is interoperable with TypeScript and enables developers to access JavaScript libraries, like Opa. Additionally, it will address the problem of boilerplate code using the same approach as Wasp. In this chapter, we will outline the requirements for a multitier language for developing web apps that might be appealing to developers.

\begin{figure}[H]
  \centering
  \includegraphics[width=0.6\textwidth]{source/figures/stars-vs-period.pdf}
  \caption{The stars of each GitHub repository over time.}
  \label{fig:github-stars}
\end{figure}

**R1. Support multitier programming**

The DSL must enable developers to write web applications in a single program without thinking about the client-server distinction. Data models, database queries, and user interfaces should all be defined in the same compilation unit. Although developers can use server resources directly on the client, the syntax of the language should still define a clear distinction between client and server resources so that no secrets are accidentally revealed. The language will be compiled and checked for errors across all tiers at compile time. This will prevent problems like *impedance mismatch* and thus limit runtime errors.

**R2. Allow developing full-stack apps declaratively**

The DSL must allow developers to specify application features using declarative syntax. Declarative programming allows developers to express the desired outcome of an operation or process rather than specifying how to achieve that outcome. For common full-stack web application functionality such as forms, routing, authentication, and permissions, the DSL must provide high-level abstractions and components. The declarative syntax must be intuitive and simple to learn, allowing developers to build applications quickly without wasting time and effort on implementation details. This declarative approach was inspired by the Wasp model, which demonstrated great potential for avoiding boilerplate code.

The DSL will help increase productivity by eliminating the need for boilerplate code and improving the maintainability of the application code by allowing developers to develop full-stack web applications declaratively. Furthermore, it will eliminate the need for previous multitier languages' client-server annotation method, which was used to allow the compiler to distinguish between the client and the server. The annotation method is typically error-prone and makes debugging the code difficult. By using declarations, the compiler will be able to differentiate between client and server code, the codebase will remain readable for developers, and the developer experience will be enhanced.

**R3. Compile to human-readable and type-safe TypeScript code using existing tech stacks**

The DSL must provide a compiler that can generate human-readable and type-safe TypeScript code. This means that the compiler must be able to translate the DSL's declarations and high-level abstractions into low-level TypeScript code that developers can easily understand and modify. To compile well-structured apps, the DSL must make use of existing tech stacks. The generator of the compiler will only compile to a NodeJS server and a ReactJS client because this is the most used stack in the world according to the "Stack Overflow Developer Survey 2022"[@stack-overflow-developer-survey-ref]. Even though the DSL must compile to NodeJS and ReactJS, it should be framework-agnostic. This means that the compiler should be able to compile to any currently available tech stacks.

By compiling existing tech stacks to human-readable and type-safe code, developers can take the compiled code and further develop it without using the DSL. Because the declarative approach with high-level abstractions that we are attempting to achieve can sometimes be limiting, it is necessary to provide developers with the ability to easily opt out of developing their app using the DSL.

**R4. Real-time by default**

The compiled app must include real-time functionality by default, allowing developers to create applications that can update and display data in real-time without writing additional code or configurations. This means that the application must receive and display data and state updates as they occur, without the need for a manual refresh of the page. This functionality can be accomplished through the use of technologies such as WebSockets, long/short polling, or SSE.

By making the compiled app have real-time functionality by default, the DSL will increase the interactivity and responsiveness of the app, providing a more seamless and engaging user experience. This is also beneficial for applications that require frequent updates, such as chat apps and collaborative tools. Furthermore, it will save developers time and effort when implementing real-time functionality and ensure that the app meets the expectations of users who expect highly interactive apps.

**R5. Allow interoperability with TypeScript**

The DSL must be interoperability with TypeScript, a popular statically-typed superset of JavaScript. This will enable developers to write custom code and extend the DSL's functionality. Because of the interoperability with TypeScript, developers will be able to take advantage of the rich ecosystem of third-party libraries and tools available in the JavaScript/TypeScript ecosystem.  The JavaScript/TypeScript ecosystem is one of the largest in the world of web development, with thousands of libraries and tools available to help developers build web applications. By making the DSL interoperable with TypeScript it should avoid the lack of community support that affected previous multitier languages.
