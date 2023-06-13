\setcounter{page}{1}
\chapter{Introduction}
\label{chap:intro}
\pagenumbering{arabic}
\singlespacing

We propose a new multitier domain-specific language (DSL) called **Thoth** that enables developers to build the database, server, and client tiers of web applications as a single, mono-linguistic program. Thoth enables developers to create web apps without writing excessive boilerplate code by offering a set of declarations that abstract most of the low-level details of web development. Multitier development is a better approach for developing web applications because with the currently available tools, developers have to build the different parts of the application using different tools and programming languages. This separation can cause a problem best known as the _impedance mismatch_ problem, as we are going to explain in section @sec:impedance-mismatch-problem. The declarative approach that our language implements has never been implemented by other multitier languages before, as we will see later in this chapter. Using the DSL, developers can define data models, queries, and interactive user interfaces, as well as implement common web features like authentication and authorization seamlessly.

In addition, Thoth compiles to real-time applications by default using server-sent events (SSE). It compiles to a NodeJS server and a ReactJS client, but it is designed to be framework-agnostic. This means that the compiler can be easily modified to compile to other frameworks. Thoth is also interoperable with TypeScript, which allows developers to benefit from the JavaScript/TypeScript ecosystem. The declarative nature of the language can be limiting, which is why we focused on creating a way for developers to easily customize the DSL declarations. Additionally, we wanted to enable them to take the compiled application and further develop it without using the DSL. That is precisely why we designed the language to compile to easily understandable human-readable code. We will discuss how we achieved these goals in greater detail in the following chapters.

## Background

Modern web apps are typically built using the 3-tier architecture. In this architecture, the application is divided into three layers, each with its own responsibility and functionality (Figure @fig:3-tier-architecture). The client tier is the first tier of the architecture, and it is in charge of rendering the user interface and handling user interactions. This tier includes the client application, which runs on the user's browser and is typically built with HTML, CSS, and JavaScript. The client app sends HTTP requests to the backend API and receives responses in JSON, XML, or HTML format. The logic tier is the architecture's second tier. This tier consists of a backend API, which is typically written in a programming language such as Java, Python, Go, or others. It is in charge of processing client requests and responding to them. It is also in charge of handling data validation, authentication, and permissions, as well as communicating with the database to perform create, read, update, and delete (CRUD) operations on the data. The database tier is the third tier of the architecture, and it is in charge of storing and managing data. This tier consists of a database running on the same or a separate machine from the server. It is created with a database management system such as MySQL, PostgreSQL, or MongoDB, which handles data storage, retrieval, and manipulation while ensuring data consistency and integrity.

In summary, the lifecycle of most web apps works as follows: whenever a user interacts with the client app, the app sends an HTTP request to the backend API. The backend API receives and processes the request, then sends a response back to the client app. Finally, the client app renders the response on the screen for the user.

![A diagram representing the 3-tier architecture.](source/figures/3-tier-architecture.pdf "3-tier architecture"){#fig:3-tier-architecture width=600}

### Impedance Mismatch Problem

However, the separation in layers that the 3-tier architecture provides can help make the development of full-stack apps more modular because each layer can be developed and maintained independently. This can cause a problem known as _impedance mismatch_. In general, impedance mismatch happens when two components in a system have different data models, data structures, or data representations. This can lead to communication problems, which can cause unexpected errors in the system and prevent it from functioning correctly.

For instance, suppose the client app has an HTML form that sends the wrong data representation to the server. As shown in Figure @fig:impedance-mismatch-1, the client app sends the age to the server as a string, but the server expects the age as an integer. In this case, the server will fail to handle the HTTP request coming from the client, leading to unexpected results. Similarly, this can happen between the server and the database when the server implements the wrong data models for the tables in the database. As shown in Figure  @fig:impedance-mismatch-2, the server is trying to select a field called `email` from the `user` table, but the user table in the database does not have a field called `email`.

\begin{figure}[H]
\centering
\begin{subfigure}{0.45\textwidth}
\centering
\includegraphics[width=\textwidth]{source/figures/impedance-mismatch-1.pdf}
\caption{}
\label{fig:impedance-mismatch-1}
\end{subfigure}
\hspace{1cm}
\begin{subfigure}{0.34\textwidth}
\centering
\includegraphics[width=\textwidth]{source/figures/impedance-mismatch-2.pdf}
\caption{}
\label{fig:impedance-mismatch-2}
\end{subfigure}
\caption{Impedance mismatch problem.}
\label{fig:impedance-mismatch}
\end{figure}

### Boilerplate Code Problem

Building full-stack apps requires too much boilerplate code, which makes the development time-consuming. Boilerplate code in full-stack web applications refers to the repetitive code that must be written. This code is often necessary for the app to function correctly but does not directly contribute to the application's unique features or functionality. This includes code for writing input validation schemas, setting up the middlewares that are used for data validation, etc. Moreover, most modern web apps usually have common features like authentication and permissions, which can be tedious to setup using the currently available tools. Here are some examples of common boilerplate code:

1- **Validation schemes**: Validation schemas can be used by validation middlewares to validate if the client app sent the right data or not. Listing \ref{lst:validation-schema-example} shows a schema that validates that clients sent a JSON object with fields `username` and `password` of type string. When the client fails to send any of the required fields or sends the incorrect data types, this schema will throw an error. The code uses a TypeScript validation library called Zod[@zod-ref].

\newpage

\begin{lstlisting}[caption=Validation schema example.,label={lst:validation-schema-example},language=JavaScript]
/\*_ An example in NodeJS/ExpressJS and Zod _/
import { z } from 'zod';

export const signup = z.object({
data: z.object({
username: z.string({
required_error: '`username` is required',
}),
password: z.string({
required_error: '`password` is required',
}),
}),
});
\end{lstlisting}

2- **Middlewares**: Middlewares are used to perform extra operations on the request before handing it to the function that is responsible for handling it. Middlewares are usually used to do data validation or check user permissions. The code in Listing \ref{lst:middleware-example} shows how middlewares can be used for data validation.

\begin{lstlisting}[caption=Middlewares example.,label={lst:middleware-example},language=JavaScript]
/\*_ An example in NodeJS/ExpressJS _/
const express = require('express');
const app = express();

/\*\*

- A function that uses the validation schema
- and validates the body of the request
  _/
  function validationMiddleware(req, res, next) { /_ body \*/ }

/\*\*

- A function that handles the requests
  _/
  function signUpUser (req, res, next) { /_ body \*/ }

/\*_ Route _/
app.post('/api/auth/signup', validationMiddleware, signUpUser);
\end{lstlisting}

3- **Simple CRUD**: Simple CRUD queries require routing, request handling, and validation schema. Instead of just writing the database query, developers will have to implement that for most API endpoints. For example, if the API endpoint just implements a simple read query that reads all records from a table in the database, it has to be implemented with all the boilerplate around it to be functional.

4- **Implementing features twice**: Some features have to be implemented twice Like authentication and validation. For example, if some content in the app requires authentication to be viewed by the users, the authentication part of the app has to be implemented once on the client-side and again on the server-side. Also, validation schemas have to be implemented twice for validating forms on the client and validating the data sent by the client on the server.

## Related Work

There are existing solutions in the literature that try to solve the problems that we have introduced. In this section, we are going to talk about the approaches of each one and their drawbacks.

### Multitier Programming

In recent years, there has been a growing interest in multitier programming in literature. Several languages have been proposed to allow for the development of full-stack web applications in a single language without the client-server distinction. Multitier languages are beneficial for developing web apps because developers can write a complete web app without thinking about the client-server distinction, and the compiler splits the code as needed and takes care of all the communication. Here are some of the languages that have introduced novel approaches to developing full-stack web apps:

\newpage

**Links**

Links[@links-ref] is a statically-typed functional language that offers developers a solution to create web apps in a single language. It achieves that by requiring the developers to annotate their code with "client" or "server" annotations on functions levels so that the compiler can split the code. Any app built using Links is compiled to an OCaml server, a client in JavaScript, and database queries in SQL. Although Links provides a type-safe web programming model, which reduces the risk of errors and vulnerabilities in the application, it still has several problems that prevent it from being widely adopted. It is a functional programming language and has unfamiliar syntax, which can make it difficult to learn for developers who are not familiar with this paradigm.

**Opa**

Similar to Links, Opa[@opa-ref] is a statically-typed functional language that offers the same features as Links and also requires the code to be annotated. Opa apps are compiled into a NodeJS server, a JavaScript client app, and a MongoDB database. It automates many aspects of modern web application programming, such as Ajax/Comet client-server communication, event-driven and non-blocking programming models. Opa did not have the same issues as Links because it used syntax similar to JavaScript, which many developers are familiar with. Moreover, Opa allows developers to reuse existing JavaScript libraries by enabling them to interact with external JavaScript code. Additionally, Opa's standard library includes the popular UI library jQuery, further enhancing its capabilities for web application development. This made it easy for developers to learn Opa and adopt it.

**Hop**

Unlike Links and Opa, Hop[@hop-ref] is a dynamically-type language. It is one of the first multitier languages introduced to solve the problem of developing full-stack apps in a single code that runs on the client and the server. The language has native support for HTML, server-side web workers, and WebSockets. Unlike Links and Opa, the language syntax is similar to JavaScript and allows developers to specify server-client annotations on an individual expression level. Developers use the `service` keyword to define services, which are essentially JavaScript functions that are automatically invoked when an HTTP request is received. Unlike Links and Opa, developers can use the `~{` marker within the service to switch from server-side to client-side context. For example, in the code snippet in Listing \ref{lst:hop-example}, `alert("world")` will run on the client while the rest will run on the server.

\begin{lstlisting}[caption=Hop example.,label={lst:hop-example},language=JavaScript]
service hello() {
return (

<html>
<div onclick=~{ alert("world") }>hello</div>
</html>
);
}
\end{lstlisting}

### No-boilerplate Programming

Frameworks are typically used to build each layer of a web app. To build client apps, for example, developers typically use JavaScript UI frameworks such as ReactJS or Angular. Similarly, they build backends with frameworks such as Django, Flask, or ExpressJS. Using any of the tools mentioned reduces the amount of boilerplate code that must be written. However, there is still boilerplate code required to be written, such as writing configuration files for both the client and server apps, as well as extra code such as the examples mentioned previously.

**Wasp**

Wasp[@wasp-ref] appeared in 2020, offering a novel approach to developing full-stack apps without boilerplate code. Its goal is to enable developers to describe the common features required by most web apps and then write the rest in ReactJS for the client app, NodeJS for the server app, and Prisma for the database. Wasp accomplishes this through the use of declarations. Wasp declarations allow developers to write what they want rather than how they want it. Wasp is only a boilerplate prevention solution; it is not a multitier language like Links or Opa.
