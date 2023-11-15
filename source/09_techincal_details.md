\chapter{Design and Implementation Details}
\label{chap:impl-details}

In this chapter, we'll take a look on how we managed to compile to human-readable code and how the compiler's type checker helps preventing the impedance mismatch problem from happening. Additionally, we will explore the architecture of the compiled application in detail, examining how it is structured.

## Compiler Architecture

The compiler architecture consists of three components: the frontend, AppSpecs generator, and code generator, as presented in Figure @fig:compiler-architecture. Each of these components will be discussed in detail in the following sections. For further information on the source code of the compiler, please refer to Appendix [2](#appendix-2-source-code).

![A diagram representing the compiler architecture.](source/figures/compiler-architecture.pdf "compiler architecture"){#fig:compiler-architecture width=900}

**Frontend**

The frontend of the compiler is responsible for two main tasks: parsing the code and constructing an AST, as well as building a symbol table using the parsed AST. Once the symbol table has been built, the frontend conducts type-checking to ensure that the code is semantically correct. Since the DSL does not have type inference feature, it requires type annotations to be included in the code. The DSL implements a nominal type system this comes as a downside for being interoperable with TypeScript. To prevent errors that could result in impedance mismatch within the application, the compiler performs a series of checks. These checks include but are not limited to:

- **UndefinedError**: This is a type of error that can occur when attempting to use an unbound declaration in general. For example, this error can occur when creating a `Create` component that specifies a form input for an undefined field in its `actionQuery`. Additionally, it can arise when the code is attempting to retrieve a non-existent field in a type that the query returns in the `FindMany` and `FindUnique` components. The purpose of this error is to alert developers that a particular tier of the application is trying to use an undefined resource from another tier or use an undefined declaration. For instance, the code shown in Listing \ref{lst:undefined-error-example} will raise this error because the `getExamples` query is trying to filter the `Example` model using an undefined field `title`.

\begin{lstlisting}[caption=UndefinedError example.,label={lst:undefined-error-example},language=Thoth]
model Example {
  id          Int    @id
  name        String
  anotherName String
}

@model(ExampleModel)
query<FindMany> getExamples {
  search: [ title ]
}
\end{lstlisting}

- **QueryTypeError**: When a component receives a query of the wrong type, the compiler will raises this error. For instance, `Create` components always expect a query of type `Create` or `Custom` with an HTTP method of post. This error helps developers ensure that their code adheres to the expected query types for each component. For example, the code shown in Listing \ref{lst:query-type-error-example} will cause an error because the `ExamplesList` component is trying to fetch examples using a query of type `Create`.

\begin{lstlisting}[caption=QueryTypeError example.,label={lst:query-type-error-example},language=Thoth]
@model(ExampleModel)
query<Create> createExample { /** body */ }

component<FindMany> ExamplesList {
  findQuery: createExample(),
  ...body...
}
\end{lstlisting}

- **RequiredFormInputError**: The compiler throws an error when the code of a `Create` or `Update` component does not specify an input for a required field in their `actionQuery`. For instance, if the `actionQuery` of a `Create` component requires two fields but the component only provides input for one of them, the compiler will raise an error. This error alerts developers to the missing input and helps ensure that their code correctly implements all required fields for the query. The code presented in Listing \ref{lst:required-form-input-error-example} will raise this error because the `ExampleCreateForm` does not implement inputs for all the fields required in the `createExample` query.

\begin{lstlisting}[caption=RequiredFormInputError example.,label={lst:required-form-input-error-example},language=Thoth]
@model(ExampleModel)
query<Create> createExample {
  data: {
    fields: [ name, anotherName ]
  }
}

component<Create> ExampleCreateForm {
  actionQuery: createExample(),
  formInputs: {
    name: { /** body */ }
  }
}
\end{lstlisting}

- **FormInputTypeError**: This error occurs when the input of a form in a `Create` or `Update` component attempts to implement the wrong input type for a specific field. For example, if a data model has a field named `title` of type `String`, but the create form for the query that uses this data model implements an input of type `NumberInput`, the compiler will raise this error. The purpose of this error is to alert developers to the input type mismatch and ensure that the form inputs are correctly implemented to match the expected field types. The example in Listing \ref{lst:form-input-type-error-example} will arise this error because the form is trying to implement an input of type `NumberInput` for the field `name` of `ExampleModel` model implemented in Listing \ref{lst:undefined-error-example} which is of type `String`. Instead, this field should be of type `TextInput`.

\begin{lstlisting}[caption=FormInputTypeError example.,label={lst:form-input-type-error-example},language=Thoth]
component<Create> ExampleCreateForm {
  actionQuery: createExample(),
  formInputs: {
    name: {
      input: { type: NumberInput }
    }
  }
}
\end{lstlisting}

By using these checks on the code, we prevent the impedance mismatch problem from occurring, and we also ensure that related runtime errors do not occur. For example, the app will not have an HTML form on the client that submits the wrong data to the server or a query on the server that tries to fetch or create a record from a data model using undefined fields or incorrect types. These checks help ensure that the app functions correctly without unexpected errors that can lead to poor user experience or application failures. There are many more error that the language's compiler can handle, please refer to Appendix [2](#appendix-2-source-code) to learn more about them in the compiler's source code.

**AppSpecs Generator**

The AppSpecs generator uses the AST and symbol table to collect information about the database tables, backend API, and client app. Based on this information, it generates three different generic intermediate representations (IR), one for each part of the app. This IR contains common information extracted and derived from the code.

The IR for the backend API includes all the necessary information to create controllers, routes, and validators. Each endpoint in the API requires a controller function that handles the client request by executing a database query, a route that maps the request to the function, validation schemas, and middlewares to check permissions and validate data. The information contained in the IR is generic and can be used with any backend to generate code for nearly any framework.

For example, the IR for the query implemented in Listing \ref{lst:find-unique-query-ir}, is represented in Figure @fig:route-ir. The AppSpecs generator uses the query definition to derive relevant data to construct the various parts of the IR. The route specifications are derived by determining the query's type, such as `FindUnique`, which indicates that the endpoint can be accessed using the GET HTTP method. Additionally, other information, such as the path, parameters, and required middleware, is derived from the query definition. Similarly, the controller function specifications are created to contain information about the inputs the query requires and other relevant details similar to the route specification. The validator specification includes information about the fields used in the query and their types. These details are later used to create validation schemas and generate type-safe backend APIs.

In addition, the AppSpecs generator creates IR for UI components and pages that can be used to compile the client and for data models to create database tables using the same technique. The generator identifies critical information for all parts of the application and stores them in data structures similar to the ones presented in Figure @fig:route-ir.

\begin{lstlisting}[caption=FindUnique query.,label={lst:find-unique-query-ir},language=Thoth]
@model(ExampleModel)
@permissions(IsAuth, OwnsRecord)
query<FindUnique> getRecordById {
  where: id
}
\end{lstlisting}

\begin{figure}[H]
  \centering
  \begin{subtable}{0.5\textwidth}
    \centering
    \begin{tabular}{|c|c|}
      \hline
      \textbf{query\_id} & getRecordById \\ \hline
      \textbf{path} & example-model \\ \hline
      \textbf{params} & (int, id) \\ \hline
      \textbf{http\_method} & GET \\ \hline
      \textbf{middlewares} & [IsAuth, OwnsRecord] \\ \hline
    \end{tabular}
    \subcaption{Route specifications.}
  \end{subtable}

  \hspace{0.05\textwidth}

  \begin{subtable}{0.5\textwidth}
    \centering
    \begin{tabular}{|c|c|}
      \hline
      \textbf{query\_id} & getRecordById \\ \hline
      \textbf{query\_type} & FindUnique \\ \hline
      \textbf{model\_name} & exampleModel \\ \hline
      \textbf{where} & true \\ \hline
      \textbf{search} & false \\ \hline
      \textbf{data} & false \\ \hline
      \textbf{owns\_record} & true \\ \hline
    \end{tabular}
    \subcaption{Function specifications.}
  \end{subtable}

  \hspace{0.05\textwidth}

  \begin{subtable}{0.5\textwidth}
    \centering
    \begin{tabular}{|c|c|}
      \hline
      \textbf{id} & Int \\ \hline
    \end{tabular}
    \subcaption{Validator specifications.}
  \end{subtable}
  \caption{Endpoint IR example.}
  \label{fig:route-ir}
\end{figure}

**Code Generator**

To generate the code, the code generator takes the IR generated in the previous step and applies it to the pre-written templates. The templates define the structure and logic of the generated code. The code generator is designed to be framework-agnostic, which means that it can generate code for any framework or library as long as the appropriate templates are available.

The code generator uses a template engine to merge the IR with the templates and produce the final code. The code in Listing \ref{lst:func-temp-example}, represents a simple template similar to the one implemented in the compiler using Jinja[@jinja-ref]. This template can be used to compile a NodeJS/ExpressJS controller function. In the example, we use the `query_id` as a name for the controller function. In the function, there is an object called `payload`, which is an object that stores the data sent in the HTTP request. It gets constructed based on the entries that the query implements. The pre-written templates include a middleware that is used to parse and validate the data in the HTTP request before executing the controller function, then it stores the result in the `req` object. We use the object `validatedPayload` to reconstruct a new payload object that can be used with the Prisma query. After that, we use the `prismaClient` to create a query based on the `model_name` and the `query_type`. Eventually, the function returns a response if the success or an error otherwise. Using this technique, we were able to compile human-readable code. This technique is used to compile all the parts in the app from the server to the client.

\newpage

\begin{lstlisting}[caption=Controller function template example.,label={lst:func-temp-example},language=JavaScript]
export const {{ query_id }} = async (req: Request, res: Response, next: NextFunction) => {
  try {
    const payload = {
      {% if where or search %}
      where: {
        {% if where %}
        ...req.validatedPayload?.where,
        {% endif %}

        {% if search %}
        ...req.validatedPayload?.search,
        {% endif %} },
      {% endif %}

      {% if data %}
        data: req.validatedPayload?.data,
      {% endif %}
    };

    const result =  await prismaClient.{{ model_name }}.{{ query_type }}(payload);

    res.status(httpStatus.OK).json(result);
  } catch (error) {
    next(error)
  }
}
\end{lstlisting}

## Compiled App

As previously mentioned, the compiler compiles to a NodeJS server, a ReactJS client, and a Prisma Schema, which is used by Prisma ORM. In this section, we will delve into each component of the application, discussing its functionality and design decisions.

**Database**

The compiler does not compile directly to SQL but instead, it compiles to Prisma schema and Prisma queries. We are using Prisma ORM because it provides a way to create type-safe queries. The query builder API generates type-safe queries that are statically checked at compile-time, reducing the risk of runtime errors and making it easier to catch errors early in the development process. One of the main features that the DSL provides is compiling to human-readable code so that we can give developers the ability to take the compiled code to further develop it without using the DSL. That is why we are using Prisma because it let us make sure that the compiled app is well-typed. Also, Prisma provides an easy way to do database migration and data seeding. For example, when a table is updated with a new column, Prisma takes care of creating the new column and filling it with data if needed. The Prisma client is configured to use Postgresql database management but it can be edited easily to use any other relational database supported by Prisma.

**NodeJS Server**

The compiler creates a representational state transfer (REST) API, which is a popular architectural style for web development. REST APIs have been widely adopted by various industries and are used in real-world applications such as social media platforms. They enable access to resources over HTTP, using methods such as GET, POST, PUT, and DELETE to perform operations on resources, such as retrieving, creating, updating, and deleting them.

One of the key benefits of REST architecture is that the server is stateless, meaning it does not store information about previous client interactions or maintain an internal state. This helps in maintaining the reliability and availability of the server because it becomes easier to replicate and distribute the server across multiple machines. This, in turn, helps to prevent server crashes and outages. Additionally, statelessness enables easy scaling, as the server does not store any state for the clients. To scale the API, we can deploy it on multiple servers and implement a load-balancer to control traffic across them.

Due to the stateless nature of the server, authentication and authorization details are not stored on the server. Instead, the client sends authentication credentials with each request. We have implemented authentication using JSON Web Tokens (JWT), which generates a compact, self-contained token containing claims that identify the user and their permissions. After logging in or requesting access to a protected resource, the server returns the JWT to the client, which stores it in local storage for subsequent requests.

The DSL is designed to produce a type-safe REST API. This means that when a request is received from a client, the API verifies that the data is valid based on the specified data types and structures. To handle invalid data types or structures, the API includes various error-handling mechanisms that notify the client of the issue. For example, if the client sends incomplete data, the server returns an error message that specifies the nature of the issue, as shown in Listing \ref{lst:errors-example}. This error can also be triggered if the client sends data of an incorrect type, such as an integer value for a field that requires a value of type string. This feature is crucial because the server API can be accessed remotely, and requests can be sent without using the official client application. By including this type-safe feature, the API ensures that only valid data is received and processed.

\begin{lstlisting}[caption=Errors example.,label={lst:errors-example},language=JavaScript]
// Returned when the client sends data with missing fields
{
  "status": 400,
  "code": "bad_request",
  "error": "invalid_input",
  "message": "Invalid inputs",
  "details": [
    {
      "code": "invalid_type",
      "parameter": "title",
      "message": "`title` is required"
    }
  ]
}

// Returned when the client sends data of the wrong type
{
  "status": 400,
  "code": "bad_request",
  "error": "invalid_input",
  "message": "Invalid inputs",
  "details": [
    {
      "code": "invalid_type",
      "parameter": "title",
      "message": "Expected string, received number"
    }
  ]
}
\end{lstlisting}

**ReactJS Client**

Finally, the compiler creates a ReactJS single page application (SPA), a type of web application that enhances the user experience by dynamically updating content on a single page without requiring multiple pages or complete page refreshes. Over the past decade, SPAs have gained popularity, especially with modern web development frameworks such as React, Angular, and Vue. Initially, when a user accesses an SPA, the server sends the HTML, CSS, and JavaScript files to the browser, and then the JavaScript code takes over and interacts with the server through the backend API to retrieve or modify data without a full-page reload. The primary advantage of SPAs is their ability to reduce the server load by only sending the initial files, resulting in faster load times and more responsive user interfaces. Additionally, SPAs offer a native application-like feel, providing more interactive and engaging user interfaces. Similar to the server, the client app includes a validation layer on HTML forms, alerting users of any incorrect data entry before making a request to the server.

**Real-time Apps by Default**

Real-time web applications require a constant flow of data between the client and the server. There are different techniques that can be used to achieve this feature. SSE allows the server to send real-time updates to the client using a single, long-lived connection. WebSockets provide a bidirectional communication channel between the client and server, allowing for real-time data exchange. Long polling involves the client making a request to the server and waiting for a response, keeping the connection open until new data is available. Short polling, on the other hand, involves the client regularly polling the server for new data.

Polling is not as practical as SSE and WebSockets because it can result in a high volume of unnecessary network traffic and latency issues, especially when there are many clients. With polling, the client sends requests to the server at regular intervals, regardless of whether new data is available. This can cause a large amount of data to be transmitted, even if there is no new information to send, which can result in increased network traffic and slower response times.

In contrast, SSE and WebSockets use a push-based approach where the server sends updates to the client only when new data is available. This eliminates the need for the client to repeatedly request new data, reducing the amount of network traffic and improving response times. Additionally, SSE and WebSockets both allow for real-time updates, making them more suitable for applications that require real-time data exchange.

After careful consideration, we opted to use SSE over WebSockets for the following reasons:

- **Simplicity**: SSE is simpler to implement than WebSockets, making it a good choice for applications that do not require bidirectional communication. SSE uses a simple HTTP connection to send data from the server to the client, while WebSockets require a dedicated WebSocket connection to enable bidirectional communication.

- **Automatic reconnection**: SSE supports automatic reconnection, which means that if the connection is lost, the client will automatically reconnect to the server without any user intervention. This makes SSE more reliable than WebSockets in some cases, as WebSockets may require additional code to handle reconnection.

- **Lightweight**: SSE is a lightweight protocol that requires less overhead than WebSockets, making it a good choice for applications that need to conserve bandwidth or have limited server resources.

To make the compiled app real-time using SSE, the client sends an initial request to the server when a page requiring data is mounted. This request loads all the necessary data and then caches it locally. After that, the client establishes a persistent connection with the server to listen for any updates to this data. Whenever there is a mutation on the server, the server sends an event to all connected clients containing the updated data. Clients then update their local state using this event, ensuring that their data is up-to-date with the server. This allows for real-time updates to be pushed to clients without the need for the client to repeatedly poll the server for new data.

## Syntax Highlighting

The DSL has a Visual Studio Code (VSCode) extension that provides syntax highlighting. This feature is crucial for a positive developer experience because it makes the language's syntax more readable and understandable. Syntax highlighting provides visual cues for different parts of the code, such as keywords, variables, and comments. It also makes it easier to spot syntax errors and other issues. The extension is implemented using TextMate grammar, which is the standard format that VSCode uses for syntax highlighting. TextMate grammar is widely used by other famous text editors like Sublime Text and Atom. This makes it easy for us to support syntax highlighting in these editors too.
